Taint and tolerance:
-------------------

- This are the mechanisms to control the pod placement on nodes.
- They prevents pods from being scheduled on to inappropriate nodes.

### Taint (on node):
- Don’t allow any pod to run on me unless the pod can handle my condition.
- Think like “Keep out” sign on a node.

### Toleration (on Pod):
- I understand that condition. I’m okay to run on that node.
- Think like a “permission slip” 

For e.g. 
1. You have one GPU node.
2. You taint it with:

gpu=true:NoSchedule  -> says “Only GPU apps allowed”

3. The Pod that need gpu adds a toleration for “gpu=true”.
4. That pod can now run on the GPU node.
5. Other pods wont run there unless they also tolerate the taint.


#### Add Taint node:
```
kubectl taint nodes node1 key1=value1:NoSchedule
```

#### Remove taint node:
```
kubectl taint nodes node1 key1=value1:NoSchedule-
```

#### Set toleration for pods in podspec:
```YAML
tolerations:
- key: "key1"  
operator: "Equal"  
value: "value1"  
effect: "NoSchedule"
```

The node controller automatically taints a node when certain conditions are true. The following taints are built in.
```
node.kubernetes.io/not-ready
node.kubernetes.io/unreachable
node.kubernetes.io/memory-pressure
node.kubernetes.io/disk-pressure
node.kubernetes.io/pid-pressure
node.kubernetes.io/network-unavailable
node.kubernetes.io/unschedulable
node.cloudprovider.kubernetes.io/uninitialized
```

### Get list of taints available on master node:
```
kubectl get nodes -o json | jq '.items[] | select(.metadata.labels["node-role.kubernetes.io/master"]=="" or .metadata.labels["node-role.kubernetes.io/master"]=="true") | {name: .metadata.name, taints: .spec.taints}'
```

### For scheduling nodes on master need to remove the taint:
```
kubectl get nodes
kubectl describe node <node-name>
```

### Remove taint:
```
kubectl taint nodes innmi1srh1-p039 node-role.kubernetes.io/control-plane-
```

If wanted to re-add taint: (just for information)
```
kubectl taint nodes innmi1srh1-p039 node-role.kubernetes.io/control-plane=:NoSchedule
```
