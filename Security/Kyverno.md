Benefits of Using Kyverno
----------------------------
### 1️⃣ Kubernetes-native (no new language)
- Policies are pure YAML
- Look and feel like normal K8s manifests
- Easy for DevOps, SREs, and platform teams

👉 No Rego, no DSL learning curve.

### 2️⃣ Strong security enforcement (by default)
- Kyverno enforces rules before bad resources enter the cluster.
#### Examples
- Block privileged containers
- Force runAsNonRoot
- Prevent latest image tags
- Enforce Pod Security Standards (PSS)

🛑 Problems are stopped at admission time, not after incidents.

### 3️⃣ Auto-fix misconfigurations (Mutation)

- This is a BIG differentiator.
- Auto-add labels & annotations
- Inject resource limits
- Add securityContext defaults
- Add node selectors / tolerations

➡️ Developers don’t need to remember everything.

### 4️⃣ GitOps-friendly policies
- Policies are Kubernetes objects
- Versioned in Git
- Reviewed via PRs
- Applied using ArgoCD / Flux

📦 Policies become infrastructure-as-code.

### 5️⃣ Image & supply-chain security built in

- Verify signed container images
- Works with Cosign / Sigstore
- Prevents untrusted or tampered images

🔐 Huge win for compliance & zero-trust pipelines.

### 6️⃣ Cluster-wide consistency

#### Ensures:
- Naming conventions
- Mandatory labels
- Resource requests & limits
- Standard security posture

💡 Every namespace behaves the same way.

### 7️⃣ Reduce operational mistakes
- Prevents “works on my machine” configs
- Stops risky YAML from reaching prod
- Less firefighting for SREs

📉 Fewer outages caused by misconfigs.

### 8️⃣ Works across all Kubernetes platforms

- EKS / GKE / AKS
- On-prem
- Bare-metal
- Edge clusters

👉 One policy set, everywhere.

### 9️⃣ Excellent observability & reporting

- Policy reports
- Audit mode (warn instead of block)
- Clear violation messages

👀 You can see who broke what and why.

### 🔟 Open-source & CNCF project
- Actively maintained
- Large community
- Enterprise-ready
- No vendor lock-in

### Install:
```
kubectl create -f https://github.com/kyverno/kyverno/releases/latest/download/install.yaml
```
### Create policy ( no latest image )
```
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-latest-tag
spec:
  validationFailureAction: enforce
  rules:
  - name: no-latest
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Image tag 'latest' is not allowed"
      pattern:
        spec:
          containers:
          - image: "!*:latest"

```
apply:
```
kubectl apply -f policy.yaml
```
🚫 Rejected immediately.

