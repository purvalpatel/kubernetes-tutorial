
1. Create directory:
```BASH
mkdir rabbitmq-deployement
cd rabbitmq-deployment
```

2. nano deployment.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

Deploy changes:
```
kubectl apply -f deployment.yaml

kubectl get all
```

Update kubernetes deployment:
```
kubectl edit deployment deployment-name

kubectl apply -f deployment_config.yaml
```

### Rolling back kubernetes deployment:

List all versions using below command,
```
kubectl rollout history
```

### Rollback previous version of deployment
```
Kubectl rollout undo deployment nginx-deployment --to-version=1
```
### Rollout history can be seen by:
```
kubectl rollout history nginx-deployment
```

### Detailed history:
```
kubectl rollout history nginx-deployment --to-version=1
```

### Scale kubernetes deployment:
```
Kubectl scale nginx-deployment --replicas=5
```

### Minimum and maximum pods should be run:
```
kubectl autoscale nginx-deployment --min=5 --max=8 --cpu-percent =75
```

### Pause and resume rollout of a deployment:
```
kubectl rollout pause nginx-deployment 

kubectl rollout resume nginx-deployment 
```

### Describe:
```
kubectl describe deployment 
```

### Find assigned nodeport:
```
kubectl get svc

kubectl get services
```

### Describe pod:
```
kubectl describe pod nginx-deployment
```

### Get logs:
```
kubectl logs nginx-deployment-54b9c68f67-6czxn
```

### Login with bash in pod
```
kubectl get pods
kubectl exec -it nginx-deployment-54b9c68f67-6czxn -- bash

Kubectl -n <namespace> exec -it <pod-name> -- bash
```

### Check mounted secrets and environment variables with pod:
```
kubectl exec -it <pod-name> -n <namespace> -- printenv
```

### Remove the complete deployment:
```
kubectl delete -f deployment.yaml
```

Note : This will only delete the deloyment which is available in yaml file. But not deletes the services.<br>
So when you check with, kubectl get svc then it will show the services.<br>
And needs to delete the services manually.<br>

### Delete the service seperately:
```
kubectl delete svc rabbitmq -n nginx-deployment
```

Cleanup all services in the namespace:
Note: Be carefull with this command.
Kubectl delete all --all -n nginx-deployment
