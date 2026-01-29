Admission controller:
--------------------
Security guard + Rule checker
- Allow request
- Deny Request
- Modify request

```
kubectl apply
     |
Authentication
     |
Authorization (RBAC)
     |
Admission Controllers
     |
etcd
```

Types of Admission controllers:
1. Mutating admission controller
2. Validating admission controller

### 1. Mutating webhook admission controller
They change the requests
- Add default labels
- inject sidecars
- add resource limits
- add security contexts

List webhookconfiguration.
```
kubectl get mutatingwebhookconfigurations
```
### 2. Validating webhook admission controller

Validate and rejects.
- Block pods running as root
- Reject images using :latest
- Enforce resource limits (CPU/memory must be set)

List 
```
kubectl get validatingwebhookconfigurations
```
| Name                  | Purpose                            |
| --------------------- | ---------------------------------- |
| `NamespaceLifecycle`  | Prevent deleting system namespaces |
| `LimitRanger`         | Enforce CPU/memory limits          |
| `ResourceQuota`       | Enforce namespace quotas           |
| `PodSecurity`         | Enforce pod security rules         |
| `DefaultStorageClass` | Assign default storage class       |
| `NodeRestriction`     | Restrict kubelet permissions       |

## 3 Common ways to use it:
- Use built-in admission controllers
- Use Pod Security Admission
- Use Webhook-based controllers
  
### Use Built-in Admission Controllers (Most are already ON)
example,
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: cpu-mem-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```
Now Kubernetes rejects pods exceeding limits.

### Pod Security Admission (Very Important 🔥)
Replaced PodSecurityPolicy (PSP).
#### Three levels:
- privileged
- baseline
- restricted

Example: <br>
```
kubectl label ns dev pod-security.kubernetes.io/enforce=restricted
```
Now pods must: <br>
- Not run as root
- Not use privileged mode
- Drop dangerous Linux capabilities

Now create bad pod:
```
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      privileged: true
```
It will get rejected.

### Use Admission Webhooks
This is what teams actually use in production.

- Kayverno (https://github.com/purvalpatel/kubernetes-tutorial/blob/94c7e7c114cddb30b1619cd479db38b295550ffb/Security/Kyverno.md)
- Gatekeeper

