```bash
helm upgrade --install argocd argo/argo-cd \
  --version 5.16.1 \
  --namespace argocd-system \
  --set configs.params."server\.disable\.auth"=true \
  --set configs.params."server\.insecure"=true \
  --set server.service.type=LoadBalancer \
  --set server.service.servicePortHttp=8080 \
  --set server.service.servicePortHttps=8443
  
```
