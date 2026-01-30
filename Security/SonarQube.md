

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
cd $project-folder
sduo sonar-scanner   -Dsonar.projectKey=Microservices-POC   -Dsonar.sources=.   -Dsonar.host.url=http://localhost:9000   -Dsonar.login=sqp_acb2a6546a02df8ce1acb566d1928d89ae393f7f
```
Output will be like, <br>
<img width="1443" height="129" alt="image" src="https://github.com/user-attachments/assets/0cfc974a-fbaf-4dcd-a785-bbd2b2e6242a" />

You can check the same in browser:
<img width="1569" height="1000" alt="image" src="https://github.com/user-attachments/assets/70f79a64-de09-4833-a6e8-d44d0f77ff7f" />


Setup with Kubernetes Helm:
--------------------------
```
helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
helm repo update
helm search repo sonarqube
kubectl create namespace sonarqube
```
Create Values.yaml
```
# REQUIRED for new Helm charts
community:
  enabled: true

# REQUIRED monitoring protection
monitoringPasscode: "sonar-monitor-123"

sonarqube:
  image:
    tag: lts

  service:
    type: NodePort
    nodePort: 30090

  persistence:
    enabled: true
    size: 10Gi

  resources:
    requests:
      cpu: "500m"
      memory: "2Gi"
    limits:
      cpu: "2"
      memory: "4Gi"
```

Install:
```
helm install sonarqube sonarqube/sonarqube \
  -n sonarqube \
  -f values.yaml
```
