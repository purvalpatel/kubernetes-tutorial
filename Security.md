How to secure kubernetes cluster?
----------------------------
### 1. cluster and infrastructure security
- Restrict access to:
  - Kube-apiserver
  - etcd
- Never expose API server port to public internet.
<br>

**Restrict global access port of API server.** <br>
**Disable Anonymous auth:** <br>
```
--anonymous-auth=false
```

### 2. Authorization and Authentication
- RBAC

### 3. Pod and Container security
- Use pod security standards (PSS)
```
kubectl label ns prod pod-security.kubernetes.io/enforce=restricted
```
Restricted means: <br>
- No privileged pods
- No hostPath
- No root user

- Run Containers as non-root user.
```
securityContext:
  runAsNonRoot: true
  readOnlyRootFilesystem: true
```

🚫 Disable dangerous settings:
`privileged: true` <br>
`hostNetwork: true` <br>
`hostPID: true` <br>

### 4. Image security
- Scan images ( Trivy )
- Example of image scan with `trivy`:
```
docker run -v /var/run/docker.sock:/var/run/docker.sock -v $HOME/Library/Caches:/root/.cache/ aquasec/trivy:0.68.2 image python:3.4-alpine
```
- This will show all the vulnerabilities of the image.

- Use Trusted registry with `cosign`
```
cosign verify myimage:1.0
```

### 5. Network Security
- Network policy
  - Deny Traffic and then explicitly allow traffic.
- Service Mesh `istio`

### 6. Secret Management
- Do not store secret in Plane YAML. instead of use kubernetes secrets.
```
kubectl create secret generic db-secret --from-literal=password=xyz
```

### 7. Runtime Security
- Use Runtime security tools ( `Falco`, `Tetragon` )
- Alert if shell executed inside container

### 8. Logging, Auditing & Monitoring
📜 Enable Audit Logs
```
--audit-log-path=/var/log/kube-apiserver.log
```
Track:<br>
- Who deleted pod
- Who accessed secrets

Monitoring <br>
- Prometheus
- Grafana
- Alertmanager

### 9. Kubernetes API Hardening

Disable unused APIs

Disable legacy auth

Use admission controllers:

PodSecurity

OPA Gatekeeper

Kyverno

Example Kyverno policy:

require runAsNonRoot=true

### 10. Node Security
🔐 Harden nodes

Minimal OS (COS, Bottlerocket)

Regular patching

Disable SSH where possible
