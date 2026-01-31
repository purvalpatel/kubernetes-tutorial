Secure CICD Pipeline:
-------------------
1. Source Code Scanning (SAST)
2. Dependency / Library Scanning (SCA)
3. Secret Scanning
4. Container Image Scanning
5. Infrastructure as Code (IaC) Scanning


### 1. Static Application Security Testing
**Scans source code for:** <br>
- SQL injection
- Hardcoded secrets
- Insecure functions
- OWASP Top 10 issues

**Popular tools:** <br>
- [SonarQube](https://github.com/purvalpatel/kubernetes-tutorial/blob/286474abc5b98110c990d46de5dc8af9bee26339/Security/SonarQube.md) ⭐
- Semgrep
- Checkmarx
- CodeQL

### 2. Dependency / Library Scanning (SCA)
**Scans:** <br>
- package.json
- pom.xml
- requirements.txt
- go.mod

**Finds:** <br>
- Vulnerable open-source libraries (CVE-based)
- Popular tools
- Snyk ⭐
- OWASP Dependency-Check
- [Trivy](https://github.com/purvalpatel/docker/blob/90f6f17d09efbfb8012c6347ded406d39444a16e/Security/Trivy.md) ⭐
- GitHub Dependabot

### 3. Secret Scanning
**Finds:** <br>
- API keys
- Tokens
- Passwords
- Certificates

**Tools:** <br>
- Gitleaks ⭐
- TruffleHog
- GitGuardian

### 4. Container Image Scanning 🐳 (MOST COMMON)
**Scans Docker images for:** <br>
- OS package CVEs
- App dependency CVEs
- Misconfigurations

**Tools:** <br>
- Trivy ⭐ (most used)
- Grype
- Anchore
- Clair

### 5. Infrastructure as Code (IaC) Scanning
**Scans:** <br>
- Terraform
- Helm charts
- Kubernetes YAML

**Finds:** <br>
- Privileged containers
- Open security groups
- Missing encryption
- Bad RBAC

**Tools:** <br>
- Checkov

### Typical CICD Pipeline:
```
Code Push
   ↓
SAST (code scan)
   ↓
Dependency scan
   ↓
Secret scan
   ↓
Build Docker image
   ↓
Container scan
   ↓
IaC scan
   ↓
Deploy
```
