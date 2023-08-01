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
