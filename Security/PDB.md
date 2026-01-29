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
**PDB is the contract between Kubernetes and your application.**
