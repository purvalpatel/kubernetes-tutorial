Production ready ArgoCD Deployment without changing the Tag version in every YAML
------------------------------------------------------------------------------
### Below is the project structure:
```
|- .gitlba-ci.yaml
|- Kustomization.yaml
|- K8s
|__ - deployment.yml
```
- This is sample project.
- In gitlab-ci.yaml you dont need to change version everytime using Sed command.
- You can use `Kustomization.yaml + Argocd set` to achive this.

### .gitlab-ci.yaml
```
stages:
  - build
  # - sonar
  - argocd_sync
# Add cleanup stage or cleanup in each job
.cleanup: &cleanup
  after_script:
    # Clean Docker cache and unused containers/images
    - docker system prune -f --filter "until=24h" 2>/dev/null || true
    - docker image prune -f 2>/dev/null || true
    - docker container prune -f 2>/dev/null || true
    # Clean up other temp files
    - rm -rf /tmp/* /var/tmp/* 2>/dev/null || true

variables:
  DOCKER_HOST: tcp://docker:2375 
  DOCKER_TLS_CERTDIR: "" 
  DOCKER_DRIVER: overlay2
  DOCKER_REGISTRY: docker.xxxx.app/devops
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA   # automatically uses the short SHA of the commit

workflow:
  rules:
    # 1️Skip pipeline if commit is made by ArgoCD Image Updater
    - if: '$CI_COMMIT_MESSAGE =~ /argocd-image-updater/'
      when: never

    # # 2️Skip pipeline if commit message says "update image tag"
    - if: '$CI_COMMIT_MESSAGE =~ /update image tag/i'
      when: never

    # 3️Run pipeline only if service code changed (not k8s YAML)
    - changes:
        - README.md
    - when: never   # every other commit is ignored


build_images:
  stage: build
  image: docker:24.0.2
  services:
    - docker:24.0.2-dind
  before_script:
    - echo "User  $DOCKER_USERNAME"
    - echo "registry $DOCKER_REGISTRY"
    - echo "registry $DOCKER_PASSWORD"
    - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin $DOCKER_REGISTRY
    - apk add --no-cache sed
    - apk add --no-cache git
  script:
    - docker pull nginx:1.14.2
    - docker tag nginx:1.14.2 docker.xxxx.app/devops/devops-test:"$IMAGE_TAG"
    - echo "Push images in $DOCKER_REGISTRY"
    - docker push $DOCKER_REGISTRY/devops-test:$IMAGE_TAG

  only:
    - main   # or master

# sonarqube_scan:
#   stage: sonar
#   image:
#     name: sonarsource/sonar-scanner-cli:latest
#     entrypoint: [""]
#   variables:
#     SONAR_USER_HOME: "git-cicd-basic/.sonar"
#     GIT_DEPTH: "0"
#   cache:
#     key: sonar
#     paths:
#       - .sonar/cache
#   script:
#     - sonar-scanner
#       -Dsonar.projectKey=git-cicd-basic
#       -Dsonar.sources=.
#       -Dsonar.host.url=$SONAR_HOST_URL
#       -Dsonar.login=$SONAR_TOKEN
#   only:
    - main

argocd_sync:
  stage: argocd_sync
  image: argoproj/argocd:latest
  before_script:
    - apt-get update && apt-get install -y curl 

  script:
    - |
      argocd login $ARGOCD_HOST \
        --username $ARGOCD_USER \
        --password $ARGOCD_PASS \
        --insecure

      echo "Changing tag version"
      argocd app set devops-test \
        --kustomize-image docker.xxxx.app/devops/devops-test:$IMAGE_TAG

      echo "Syncing job"
      argocd app sync devops-test \
            --resource "apps:Deployment:nginx-deployment" \
            --async \
            --prune=false \
            --timeout 300
  only:
    - main


```

### kustomization.yaml
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - k8s/deployment.yaml

images:
  - name: docker.xxx.app/devops/devops-test
    newTag: "69ee13f0"
```

### k8s/deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: devops-test
  labels:
    app: nginx
spec:
  replicas: 2
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
        image: docker.xxx.app/devops/devops-test:0.1
        ports:
        - containerPort: 80
```

Note: <br>
- kustomization.yaml Image tag will overwrite the image tag added into `k8s/deployment.yml`.
- `argocd app set devops-test` command will overwrite the image tag.
- This tag will not change into gitlab kustomization.yaml but directly changed into deployment.

If want to add multiple images then do like this in `kustomization.yaml` <br>
```
images:
  - name: docker.xxx.app/devops/user-service
    newTag: latest
  - name: docker.xxx.app/devops/project-service
    newTag: latest
  - name: docker.xxx.app/devops/sbdd-service
    newTag: latest
```
