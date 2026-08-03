- Daemonset is like a special kind of deployment that says:
“ Run one copy of this pod on evert node ( or specific nodes ) in a cluster.

- As nodes are added to cluster Pods are added to them. As nodes are removed from cluster, those pods are garbage collected.

Typical use of daemonset:

1. Running cluster storage daemon on all node
2. Running logs collection daemon on all node
3. Running monitoring daemon like grafanna on all node.

E.g.
```
apiVersion: apps/v1
kind: DaemonSet
metadata:  
name: fluentd-elasticsearch  
namespace: kube-system
```

### Label node:
```
kubectl label nodes <node-name> gpu=true
```
