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
METALLB_CIDR=`docker network inspect k3d-${CLUSTER_NAME} | jq -r ".[0].IPAM.Config[0].Subnet" | awk -F'.' '{print $1"."$2"."$3"."128"/"25}'`

cat <<EndOfMessage | kubectl -n metallb-system apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: docker-host
spec:
  addresses:
    - "${METALLB_CIDR}"
  avoidBuggyIPs: true
EndOfMessage

cat <<EndOfMessage | kubectl -n metallb-system apply -f -
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: docker-host
spec:
  ipAddressPools:
  - docker-host
EndOfMessage
```
