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

### 1. Mutating admission controller
They change the requests
- Add default labels
- inject sidecars
- add resource limits
- add security contexts

### 2. Validating admission controller

Validate and rejects.
- Block pods running as root
- Reject images using :latest
- Enforce resource limits (CPU/memory must be set)

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
