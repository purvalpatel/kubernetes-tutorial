## Description:

- Provides identifying metadata for an object.
- Used for grouping, viewing and operating.
- Labels are key value pair.

### Create label:
```
 kubectl run alpaca-prod \
  --image=gcr.io/kuar-demo/kuard-amd64:blue \
  --replicas=2 \
  --labels="ver=1,app=alpaca,env=prod"
```

### View label:
```
kubectl get deployments --show-labels

kubectl get deployments -L canary
```

### Remove label:
You can remove label by applying - suffix.
```
kubectl label deployments alpaca-test "canary-"
```

### Label Selectors:
If we want to list only resources which have tag ver=2 then we have to use selector tag.
```
kubectl get pods --selector="ver=2"
```

### Annotations:
- Annotations provide place to store additional metadata for kubernetes object.
- Can store large unstructured data. That is extrainformation about an object. That is not used for tag or selection. Like, descriptions, who maintains.
