configmaps:
------------

- Configuration data.
- Used to externalized the application configurations from your container images.
- Change application behaviour without rebuilding images
- Share configs accross multiple pods or containers.

ConfigMap.yaml
```YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
  namespace: default
data:
  APP_ENV: production
  APP_DEBUG: "false"
  MAX_CONN: "100"
```

Ways to use configMaps:

### 1.Use like Environment variables:
```YAML
spec:
  containers:
    - name: my-app
      image: my-image
      envFrom:
        - configMapRef:
            name: my-config
```

Inside the container APP_ENV will get production.

### 2.Use like Files in a volume:
```YAML
spec:
  containers:
    - name: my-app
      image: my-image
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: my-config
```

Now container will see,<br>
**/etc/config/APP_ENV <br>
/etc/config/APP_DEBUG** <br>

### 3.Use specific keys as individual Env Vars:
```YAML
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: APP_ENV
```

#### Create configMap from CLI
```
kubectl create configmap my-config --from-file=config.properties
```

View and Edit configmaps
```
Kubectl get configmap
```

Get the yaml of configmap.
```
kubectl get configmap -o yaml 
```
Get more details:
```
kubectl describe configmap kube-root-ca.crt
```

Edit configmap:
```
Kubectl edit configmap kube-root-ca.crt
```

🧠**Notes:**
- Configmaps **are not for the secrets**.
- Config **changes wont restart pods**. Need to manually redeploy.
- Max size **1 MB** per configmap.

Secrets:
---------

- For sensitive data.
- Password, TLS certificates, API Keys, Database Credentials.

Example:
Secret.yaml
```YAML
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
data:
  username: YWRtaW4=       # base64 of 'admin'
  password: cGFzc3dvcmQ=   # base64 of 'password'
```

Use below commands to encode and decode the password: <br>
**to encode** - echo -n 'admin' | base64 <br>
**to decode**  - echo 'YWRtaW4=' | base64 --decode <br>

### Ways to use secrets in pods:

#### 1.As Environment variables:
```YAML
env:
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: username
  - name: DB_PASS
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: password
```

#### 2.As Mounted files:
```YAML
volumeMounts:
  - name: secret-vol
    mountPath: "/etc/secrets"
volumes:
  - name: secret-vol
    secret:
      secretName: my-secret
```

Inside the pods you will have files,<br>

**/etc/secrets/username<br>
/etc/secrets/password**


#### Create secrets manually:
```BASH
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=password
```

View secret:
```BASH
kubectl get secret
```
To decode:
```BASH
kubectl get secret my-secret -o jsonpath="{.data.username}" | base64 --decode
```
