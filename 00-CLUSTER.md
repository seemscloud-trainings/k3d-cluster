```bash
export CLUSTER_NAME=seems
```

```bash
k3d cluster delete "${CLUSTER_NAME}"
```

```bash
export K3D_FIX_MOUNTS=1

k3d cluster create \
  --servers 3 \
  --agents 6 \
  --k3s-arg "--cluster-cidr=10.100.0.0/16@server:*" \
  --k3s-arg "--service-cidr=10.200.0.0/16@server:*" \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --no-lb ${CLUSTER_NAME}
  
#  --image seemscloud/k3s:v1.27.4-k3s1-nfs
```

```bash
for i in `seq 0 2` ; do
  kubectl taint nodes "k3d-${CLUSTER_NAME}-server-$i" dedicated=control-plane:NoSchedule
done

for i in `seq 0 5` ; do
  kubectl label node k3d-${CLUSTER_NAME}-agent-${i} node-role.kubernetes.io/generic=true
  kubectl label node k3d-${CLUSTER_NAME}-agent-${i} node-role.kubernetes.io/worker=true
done
```

```bash
export PROJ_PATH="seemscloud-trainings/helm-argocd-self"
export BRANCH_NAME="main"

export REPO_URL_RAW="https://raw.githubusercontent.com/${PROJ_PATH}"
export REPO_URL="https://github.com/${PROJ_PATH}.git"

kubectl create namespace argocd-system

helm upgrade \
    --install argocd argo/argo-cd \
    --namespace argocd-system \
    --version 5.51.4 \
    --values "${REPO_URL_RAW}/${BRANCH_NAME}/base/stacks/argocd/argocd/values.yaml"

helm upgrade \
    --install argocd-apps argo/argocd-apps \
    --namespace argocd-system \
    --version 1.4.0 \
    --values "${REPO_URL_RAW}/${BRANCH_NAME}/base/stacks/argocd/argocd-apps/values.yaml"

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
