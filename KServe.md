Introduction
----
- Platform for bot h Generative and Predictive AI inference on kubernetes
- Deploy,Scale and Manage ML Models In production.

### In Simple words:
- K8s Manages Containers.
- Kserve Manages ML Models.

Traditional Kubernetes Deployment contains:
- Deployment,
- Services
- Autoscales
- Rollouts
- etc.

Kserve:
- Deploy Models and make it available as an API.

### Why Kserve?
- Expose it as an API
- Scale it based on traffic
- Update to new model versions
- Roll back if something breaks
- Monitor it
- Keep it stable

### What KServe does automatically:
- Creates Deployment
- Creates Service
- Creates Autoscaler
- Exposes endpoint
- Handles scaling up/down

## Real World Use Case 1 — API Serving
- Deploy All Models
- Scale based on traffic
- Monitor Health

## Real World Use Case 2 — LLM Deployment
If serving models using vLLM. <br>
and you dont want to manage Deployment, Services, Autoscaling etc. <br>

KServe allows canary deployment. <br>

## Real World Use Case 3 — AutoScaling
With Kserve:
- Scale based on traffic
- Scale to zero ( For small models )
- Scale up automatically with traffic increases.

| Tool       | What It Does            |
| ---------- | ----------------------- |
| Kubernetes | Manages containers      |
| Docker     | Runs containers         |
| vLLM       | Runs LLM inference      |
| Triton     | Runs ML inference       |
| **KServe** | Manages model lifecycle |

## When Should You Use KServe?
Use KServe if:
- You run models in Kubernetes
- You need autoscaling
- You need version control
- You serve multiple models
- You want production-grade ML infra


