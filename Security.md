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
- Example of image scan:
```
docker run -v /var/run/docker.sock:/var/run/docker.sock -v $HOME/Library/Caches:/root/.cache/ aquasec/trivy:0.68.2 image python:3.4-alpine
```
