```bash
helm upgrade --install argocd argo/argo-cd \
  --version 5.16.1 \
  --namespace argocd-system \
  --set fullnameOverride=argocd \
  \
  --set configs.params."server\.disable\.auth"=true \
  --set configs.params."server\.insecure"=true \
  --set server.service.type=LoadBalancer \
  --set server.service.servicePortHttp=8080 \
  --set server.service.servicePortHttps=8443 \
  --set repoServer.replicas=3 \
  --set applicationSet.replicas=3 \
  --set controller.replicas=3 \
  --set server.replicas=3 \
  --set redis.enabled=false \
  --set redis-ha.enabled=true
```
