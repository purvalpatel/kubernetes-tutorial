## Phoenix Setup for LLM Model tracing:

### Setup

#### pheonix-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phoenix
  namespace: observability
spec:
  replicas: 1
  selector:
    matchLabels:
      app: phoenix
  template:
    metadata:
      labels:
        app: phoenix
    spec:
      containers:
      - name: phoenix
        image: arizephoenix/phoenix:latest
        ports:
        - containerPort: 6006
        - containerPort: 4317
        env:
        - name: PHOENIX_PORT
          value: "6006"
        - name: PHOENIX_ENABLE_OTLP
          value: "true"
        - name: PHOENIX_WORKING_DIR        # ← tell Phoenix where to persist data
          value: "/mnt/phoenix-data"
        volumeMounts:
        - name: phoenix-storage
          mountPath: /mnt/phoenix-data     # ← mount the volume
      volumes:
      - name: phoenix-storage
        persistentVolumeClaim:
          claimName: phoenix-pvc           # ← attach PVC
```
Apply : `kubectl apply -f pheonix-deployment.yaml` <br>

#### Create phoenix-pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: phoenix-pvc
  namespace: observability
spec:
  storageClassName: local-path    # ← add this
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
Apply: `kubectl apply -f phoenix-pvc.yaml` <br>

#### service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: phoenix
  namespace: observability
spec:
  type: NodePort
  selector:
    app: phoenix
  ports:
  - name: ui
    port: 6006
    targetPort: 6006
  - name: otlp
    port: 4317
    targetPort: 4317
```

#### Create Project in Phoenix:
Name: vllm-observability

#### Setup OpenTelemetry Collector:
- Setup open-telemetry-collector which send traces to phoenix.

`otel-values.yaml`
```
mode: deployment

image:
  repository: otel/opentelemetry-collector-contrib

config:
  receivers:
    otlp:
      protocols:
        grpc: {}
        http: {}

    zipkin:
      endpoint: 0.0.0.0:9411

  processors:
    memory_limiter:
      check_interval: 5s
      limit_mib: 400
      spike_limit_mib: 100

    resource/phoenix_project:
      attributes:
        - key: openinference.project.name
          value: vllm-observability
          action: upsert

    batch: {}

  exporters:
    otlp/phoenix:
      endpoint: phoenix.observability.svc.cluster.local:4317
      tls:
        insecure: true

  service:
    pipelines:
      traces:
        receivers: [zipkin, otlp]
        processors: [memory_limiter, resource/phoenix_project, batch]
        exporters: [otlp/phoenix]

service:
  type: ClusterIP

```
#### install with helm:
```
helm install otel-collector open-telemetry/opentelemetry-collector   --namespace observability   -f otel-values.yaml

## for uninstall [ for reference only.]
helm uninstall otel-collector -n observability
```

#### create istio-operator:
```
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: tracing-config
  namespace: istio-system

spec:
  meshConfig:
    enableTracing: true
    defaultConfig:
      tracing:
        sampling: 100
        zipkin:
          address: otel-collector-opentelemetry-collector.observability.svc.cluster.local:9411
```
install
```
~/istio-1.29.0/bin/istioctl install -f istio-operator.yaml
```
verify:
```
kubectl get cm istio -n istio-system -o yaml | grep tracing -A5
```

Restart:
```
kubectl rollout restart deployment otel-collector-opentelemetry-collector -n observability
```


pipeline is:
```
User
 ↓
Istio ingressgateway
 ↓
Envoy proxy span
 ↓
OpenTelemetry Collector (Zipkin receiver)
 ↓
Phoenix (OTLP exporter)
```

> This will only show the Spans, not the Response, Token and prompt.
> For getting this details we need small service i.e. application API.

<img width="1917" height="892" alt="image" src="https://github.com/user-attachments/assets/ee8a8c73-e447-47d2-a03a-f66052717f21" />


## Prompt, traces monitoring
> Currentlt the tracing mechanism is working only for network traffic, Not LLM logic.

So it shows,
- URL
- Status code
- Request/Response size
- Request/Response time
- body etc.

This data comes from the `envoy proxy`.

- But it will not show the Promt, token, this lived inside the request body.

> Istio does not parse the request body. <br>
This information is only visible to the application that actually calls the model.

```
client -> Istio ingressgateway -> Envoy proxy -> OpenTelemetry Collector -> Phoenix
```

To get this type of details, something must run code that knows the prompt and response. <br>

So we need one application that runs between the ingress gateway and vLLM. which we can call as  API Service.
```
client -> Istio ingressgateway -> API Service -> vLLM service -> vLLM Pod -> GPU
```

> Istio : networking, routing, security <br>
API Services: Prompt processing, logging, tracing <br>
vLLM : GPU inference. <br>

### Create API service

Create directory:
```
llm-api/
 ├── app.py
 ├── requirements.txt
 └── Dockerfile
```

### FastAPI gateway (app.py)

This gateway forwards requests to vLLM and sends traces.
```
from fastapi import FastAPI, Request
import requests
import os

from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

trace.set_tracer_provider(TracerProvider())

exporter = OTLPSpanExporter(
    endpoint="otel-collector-opentelemetry-collector.observability.svc.cluster.local:4317",
    insecure=True
)

trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(exporter)
)

tracer = trace.get_tracer(__name__)

app = FastAPI()
FastAPIInstrumentor.instrument_app(app)

VLLM_URL = os.getenv("VLLM_URL", "http://nucurate-model-service.vllm.svc.cluster.local:8000")

@app.post("/v1/chat/completions")
async def chat(req: Request):

    body = await req.json()

    with tracer.start_as_current_span("llm_request") as span:

        span.set_attribute("llm.model", body.get("model"))

        if "messages" in body:
            span.set_attribute("llm.prompt_preview", str(body["messages"])[:200])

        r = requests.post(
            f"{VLLM_URL}/v1/chat/completions",
            json=body
        )

        span.set_attribute("llm.status_code", r.status_code)

        return r.json()
```

requirements.txt
```
fastapi
uvicorn
requests
opentelemetry-sdk
opentelemetry-exporter-otlp
opentelemetry-instrumentation-fastapi
```

Dockerfile
```
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py .

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```
Build and push:
```
docker build -t <registry>/llm-api:latest .
docker push <registry>/llm-api:latest
```
Kubernetes deployment
``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: llm-api
  namespace: vllm
spec:
  replicas: 1
  selector:
    matchLabels:
      app: llm-api
  template:
    metadata:
      labels:
        app: llm-api
    spec:
      containers:
      - name: api
        image: <registry>/llm-api:latest
        ports:
        - containerPort: 8000
        env:
        - name: VLLM_URL
          value: http://nucurate-model-service.vllm.svc.cluster.local:8000
```
Service
```
apiVersion: v1
kind: Service
metadata:
  name: llm-api-service
  namespace: vllm
spec:
  selector:
    app: llm-api
  ports:
  - port: 8000
    targetPort: 8000
```

### Update Istio routing

Change your VirtualService so traffic goes to the API instead of vLLM directly.
```
route:
- destination:
    host: llm-api-service
    port:
      number: 8000
```

Your request flow becomes:
```
Client
  ↓
Istio
  ↓
API Gateway
  ↓
vLLM models
  ↓
GPU
```
