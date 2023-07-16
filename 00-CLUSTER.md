```bash
k3d cluster delete "${CLUSTER_NAME}"
```

```bash
export CLUSTER_NAME=seems
```

```bash
k3d cluster create \
  --servers 3 \
  --agents 6 \
  --k3s-arg "--cluster-cidr=10.100.0.0/16@server:*" \
  --k3s-arg "--service-cidr=10.200.0.0/16@server:*" \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --no-lb ${CLUSTER_NAME}


```
