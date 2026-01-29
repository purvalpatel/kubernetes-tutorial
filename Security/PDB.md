PDB ( Pod Distruption Budget ):
----------------------------
PDB exists to protect application availability during planned Kubernetes actions.

### What problem PDB really solves
Kubernetes sometimes needs to intentionally stop pods to do its job: <br>
- Draining a node
- Upgrading nodes
- Autoscaling down
- Moving workloads for maintenance

From Kubernetes’ point of view:
```TEXT
“I’m allowed to evict pods.”
```

From your app’s point of view:
```TEXT
“If too many pods go down together, users will feel it.”
```
**PDB is the contract between Kubernetes and your application.** <br>
**“When you (Kubernetes) choose to evict pods, don’t evict too many at once.”**

### When PDB is absolutely necessary
✔ Stateful apps <br>
✔ APIs with SLAs <br>
✔ Databases (with replicas) <br>
✔ Message brokers <br>
✔ Any user-facing production service <br>

### How to implement ?
- Before starting implementation decide how many pods you want to keep running ? 

Your current **Deployment**.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
```
**Replicase must be greater than your distruption limit.**

Chose Strategy. `MinAvailable`, `MaxUnavailable`
```
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```
Apply PDB:
```
kubectl apply -f pdb.yaml
```
Verify:
```
kubectl get pdb
kubectl describe pdb web-pdb
```

You can test this by draining node:
```
kubectl drain <node-name> --ignore-daemonsets
```
