```bash
export PROJ_PATH="seemscloud/helm-argocd-self"
export BRANCH_NAME="main"

export REPO_URL_RAW="https://raw.githubusercontent.com/${PROJ_PATH}"
export REPO_URL="https://github.com/${PROJ_PATH}.git"


helm upgrade \
    --install argocd argo/argo-cd \
    --namespace argocd-system \
    --version 5.42.0 \
    --values "${REPO_URL_RAW}/${BRANCH_NAME}/base/argocd/values.yaml" \
    --values "${REPO_URL_RAW}/${BRANCH_NAME}/overlays/seemscloud/values/argocd/values.yaml"

helm upgrade \
    --install argocd-apps argo/argocd-apps \
    --namespace argocd-system \
    --version 1.4.0 \
    --values "${REPO_URL_RAW}/${BRANCH_NAME}/base/argocd-apps/values.yaml" \
    --values "${REPO_URL_RAW}/${BRANCH_NAME}/overlays/seemscloud/values/argocd-apps/values.yaml"

kubectl apply -f <(kustomize build "${REPO_URL}/overlays/seemscloud?ref=${BRANCH_NAME}")
```

```bash
helm upgrade --install argo-workflows argo/argo-workflows \
  --version 0.25.1 \
  --namespace argo-system \
  --set fullnameOverride=argo-workflows \
  --set crds.install=true \
  --set crds.keep=false \
  --set controller.name=controller \
  --set controller.metricsConfig.enabled=true \
  --set controller.telemetryConfig.enabled=true \
  --set controller.serviceAccount.enabled=true \
  --set controller.logging.level=info \
  --set server.enabled=true \
  --set server.name=server \
  --set server.replicas=1 \
  --set server.secure=false \
  --set server.serviceAccount.create=true \
  --set server.logging.level=info \
  --set server.serviceType=LoadBalancer \
  --set server.servicePort=8080 \
  --set server.extraArgs[0]="--auth-mode=server" \
  --set extraObjects[0].apiVersion=v1 \
  --set extraObjects[0].kind=Secret \
  --set extraObjects[0].metadata.name=argo-workflows-minio-creds \
  --set extraObjects[0].data.accessKey=dXNlcjAx \
  --set extraObjects[0].data.secretKey=VXNlcjEyMyFAIw== \
  --set useDefaultArtifactRepo=true \
  --set artifactRepository.archiveLogs=true \
  --set artifactRepository.s3.endpoint=argo-workflows-minio:9000 \
  --set artifactRepository.s3.bucket=user01 \
  --set artifactRepository.s3.insecure=true \
  --set artifactRepository.s3.accessKeySecret.name=argo-workflows-minio-creds \
  --set artifactRepository.s3.accessKeySecret.key=accessKey \
  --set artifactRepository.s3.secretKeySecret.name=argo-workflows-minio-creds \
  --set artifactRepository.s3.secretKeySecret.key=secretKey
```
