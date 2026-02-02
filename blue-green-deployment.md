🔵 Blue → currently serving users (live) <br>
🟢 Green → new version (idle / testing) <br>

<img width="597" height="375" alt="image" src="https://github.com/user-attachments/assets/1ada44f2-d3f5-47e7-a587-1988ff35a706" />

### Where Blue-Green is BEST
✅ Critical production systems <br>
✅ Releases with DB-safe changes <br>
✅ When you want fast rollback <br>

### ❌ Not great for:
- Large DB schema breaking changes
- Long-running background jobs

### Kubernetes deplpoyment:
- Blue Deployment : `Label : blue`
- Green Deployment: `Label : green`
- Service : `version : blue` (Traffic switch key)
Just change the version : green to switch <br>

For rollback chnage key in service, `version: blue` <br>


## Database compatibility:

**Blue-Green = two application versions, not two databases.**

### ✅ Standard Production Setup (MOST COMMON) <br>
```
           Users
             |
        Load Balancer
             |
        Blue / Green App
             |
        Shared Database
```
### ⚠️ The REAL Challenge: Database Compatibility <br>

Since both versions may run briefly at the same time, DB changes must be: <br>

- **Backward Compatible** (New database changes do NOT break the old application version)

<br>

That’s the golden rule. <br>

## Deployment Manifests:

Blue Deployment (Live)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1
        ports:
        - containerPort: 8080

```


Green Deploynment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: app
        image: myapp:v2
        ports:
        - containerPort: 8080

```

Service: (traffic switch key )
```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue   # 🔵 currently live
  ports:
  - port: 80
    targetPort: 8080
```

Best Strategy: <br>
Argo Rollouts + Kubernetes Service

| Feature         | K8s Deploy | Ingress | Service Mesh | Argo Rollouts |
| --------------- | ---------- | ------- | ------------ | ------------- |
| Rolling update  | ✔          | ✔       | ✔            | ✔             |
| Canary          | ⚠️ Limited | ✔       | ✔            | ✔             |
| Blue-Green      | ✔          | ✔       | ✔            | ✔             |
| Auto rollback   | ❌          | ❌       | ❌            | ✔             |
| Metrics gate    | ❌          | ❌       | ❌            | ✔             |
| GitOps friendly | ⚠️         | ⚠️      | ✔            | ⭐✔            |
