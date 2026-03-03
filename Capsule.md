How do i safely give multiple team access to the same cluster without giving them rights of cluster administrator? <br>
Here, **Capsule** comes into the picture.

Capsule introduces new level of abstraction, called **Tenant**. <br>

**Tenant can:**
- Own multiple namespaces
- Limited cluster-wide permissions
- Quota restrictions
- Allowed storage classes
- allowed ingress classes

Instead of managing raw RBAC manually for each namespace you manage, you can use capsule.
```
tenant -> Namespace -> Policies
```
Much cleaner.

**Capsule controls:**
- Resource Quota
- Network policies
- PSS
- Allowed container registry
- Storage class

### Soft vs Hard Multi-tenancy:

1. **Hard Multi-tenancy**: Seperate cluster per team
2. **Soft Multi-tenancy**: `Capsule` healpps to achive this, single cluster, strong logical isolation.


## Capsule Setup
1. Install Capsule on Kubernetes.
```
helm repo add clastix https://clastix.github.io/charts
helm repo update

kubectl create namespace capsule-system
helm install capsule clastix/capsule -n capsule-system

```

Verify pods:
```
kubectl get pods -n capsule-system
```
2. Define Tenants:
A Tenant is a logical grouping of resources and namespaces in Kubernetes. You can enforce resource quotas, control the number of namespaces that a tenant can create, and apply other tenant-specific policies. <br>

Create Tenant (tenant-ml.yaml)
```
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata:
  name: tenant-ml
spec:
  owners:
    - name: ml-team
      kind: Group
  namespaceOptions:
    quota:
      hard:
        requests.cpu: "50"
        requests.memory: 100Gi
        limits.nvidia.com/gpu: "4"
```
Apply to cluster:
```
kubectl apply -f tenant-ml.yaml
```
Verify tenants:
```
kubectl get tenants
```

3. Assign Namespaces to Tenants
Label namespace with the tenant.
```
kubectl create namespace ml-dev
kubectl label namespace ml-dev capsule.clastix.io/tenant=tenant-ml
```
Now, the ml-dev namespace belongs to the tenant-ml tenant, and Capsule will apply the quotas, restrictions, and policies defined for that tenant. <br>

Verify labels:
```
kubectl get namespaces --show-labels
```


## Other usages:
1. Enforce storage-class
```
spec:
  namespaceOptions:
    quotas:
      hard:
        requests.cpu: "50"
        requests.memory: 100Gi
        limits.nvidia.com/gpu: "4"
    allowedStorageClasses:
      - standard
      - fast-storage
```
You can define network policies, ingress classes, and more, depending on your cluster setup. <Br>

2. Self-service namespace creation:
```
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata:
  name: tenant-ml
spec:
  owners:
    - name: ml-team
      kind: Group
  namespaceOptions:
    quota:
      hard:
        requests.cpu: "50"
        requests.memory: 100Gi
        limits.nvidia.com/gpu: "4"
    namespaceCreator: true
```
Now, members of ml-team will be able to create namespaces without requiring admin intervention. <br>

To monitor tenant usage (e.g., CPU, memory, GPU), you can use Kubernetes-native tools like Prometheus along with Capsule-specific metrics. <dr>

After creating tenants and namespaces, you need to assign roles and set up permissions to allow users (or groups) to interact with resources within the tenant. <br>

4. RoleBindings and Access Control

Role-binding.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ml-team-admin
  namespace: ml-dev
subjects:
  - kind: Group
    name: ml-team
roleRef:
  kind: Role
  name: admin
  apiGroup: rbac.authorization.k8s.io
```
`kind`: Group and name: ml-team specify that the role is granted to the ml-team group. <br>
`roleRef` defines that the group will have the admin role within the ml-dev namespace.
Apply:
```
kubectl apply -f rolebinding-ml-team-admin.yaml
```

Validate access:
```
kubectl auth can-i create deployments --namespace=ml-dev
```
