```bash
helm upgrade --install metallb metallb/metallb \
  --version 0.13.4 \
  --namespace metallb-system

helm upgrade --install cert-manager jetstack/cert-manager \
  --version v1.7.2 \
  --namespace cert-manager-system \
  --set installCRDs=true
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
