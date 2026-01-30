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
- SonarQube ⭐
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
- Trivy
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


SonarQube:
-----------
#### Install sonarQube:
```
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:lts
```

Access web UI:
```
http://<server-ip>:9000

## Default login
username: admin
password: admin

```
#### Create a Project in SonarQube
```
Login → Projects → Create Project
```
- Choose:
   - Project Key (example: my-app)
   - Display Name

- Select:
   - Locally ( Project should be there on Local as of now for manual testing )
     
- Generate token (VERY IMPORTANT)
<img width="1338" height="763" alt="image" src="https://github.com/user-attachments/assets/38e1c45f-8d5f-4272-a0a4-2410efeef365" />

📌 Save the token securely.

Setup Sonar-scanner ( Client )
https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-8.0.1.6346-linux-x64.zip <br>

```
unzip sonar-scanner-cli-8.0.1.6346-linux-x64.zip
sudo mv sonar-scanner-8.0.1.6346-linux-x64/ /opt/
sudo ln -s /opt/sonar-scanner-8.0.1.6346-linux-x64/bin/sonar-scanner sonar-scanner
```

Now you can run scans:
```
sonar-scanner   -Dsonar.projectKey=Microservices-POC   -Dsonar.sources=.   -Dsonar.host.url=http://localhost:9000   -Dsonar.login=sqp_acb2a6546a02df8ce1acb566d1928d89ae393f7f
```

