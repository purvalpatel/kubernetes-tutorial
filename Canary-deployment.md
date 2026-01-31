## What is Canary Deployment? (Simple words)
- You release a new version to a small % of users first.

If it behaves well → gradually increase traffic. <br>
If it misbehaves → rollback with minimal blast radius. <br>
```
Users
  |
Load Balancer
  |
90% → v1 (stable)
10% → v2 (canary)
```

## When Canary is BEST

✅ High-traffic systems <br>
✅ Risky changes <br>
✅ Performance-sensitive apps <br>
✅ When you want real user validation <br>

### ❌ Not ideal for:

- Very low traffic apps
- Hard DB breaking changes

### We’ll do:
```
myapp-v1 (stable)

myapp-v2 (canary)
```

**Traffic split using labels**

## Kubernetes deployment :
### 1️⃣ Stable Deployment (v1)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: app
        image: myapp:v1
        ports:
        - containerPort: 8080
```

### 2️⃣ Canary Deployment (v2)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v2
spec:
  replicas: 1   # ~10% traffic
  selector:
    matchLabels:
      app: myapp
      version: v2
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
      - name: app
        image: myapp:v2
        ports:
        - containerPort: 8080
```

### 3️⃣ Service (Traffic Split by Pod Count)
```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

### Traffic distribution:
```
9 pods v1 → 90%
1 pod v2 → 10%
```

### 4️⃣ Monitor Canary (MOST IMPORTANT)

Watch:

Error rate

Latency

CPU/memory

Business metrics (orders, logins)

kubectl logs deploy/myapp-v2
kubectl get pods

### 5️⃣ Promote Canary Gradually

Increase replicas:
```
kubectl scale deploy myapp-v2 --replicas=3
kubectl scale deploy myapp-v1 --replicas=7
```

Then:
```
30% → 50% → 100%
```

### 6️⃣ Rollback (Very Safe)
```
kubectl scale deploy myapp-v2 --replicas=0
```

Only 10% users affected at most.

## 🌐 Canary with Istio (BEST PRACTICE)

Replica-based canary is rough. <br>
Istio gives exact traffic control. <br>

VirtualService example:
```
http:
- route:
  - destination:
      host: myapp
      subset: v1
    weight: 90
  - destination:
      host: myapp
      subset: v2
    weight: 10
```

### Change weights safely.

🧪 Canary + Database (Important)

### Same rules as Blue-Green:

- ✅ Backward compatible schema
- ✅ Feature flags
- ❌ Breaking DB changes

🚀 Production Canary Flow
```
Deploy v2 (1%)
↓
Monitor metrics
↓
Increase traffic
↓
Full rollout
↓
Remove v1
```


















