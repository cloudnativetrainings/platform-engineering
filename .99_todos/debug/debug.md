
## oidc plugin

```bash
kubectl oidc-login clean
kubectl --kubeconfig=./.secrets/kubeconfig-kcp-admin.yaml get apiexport

export KUBECONFIG=./.secrets/kubeconfig-kcp-admin.yaml
kubectl ws list --kubeconfig=./.secrets/kubeconfig-kcp-admin.yaml


BUG!!!!!!!!!!!!!!!!!!!!
kubectl ws list --kubeconfig=./.secrets/kubeconfig-kcp-admin.yaml
Error: current "root" context not found
works ia export KUBECONFIG=./.secrets/kubeconfig-kcp-admin.yaml
BUG!!!!!!!!!!!!!!!!!!!!
```

## label the resourceschema

```bash
kubectl ws org
kubectl get apiexport
kubectl get apiresourceschema
kubectl label apiresourceschemas va69bce7c.myservices.org.com syncagent.kcp.io/api-group=myservice
```

## restart syncagents

```bash
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml -n kdp-system get pods
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml -n kdp-system delete pods --all
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml -n kdp-system logs -f -l app.kubernetes.io/name=kcp-api-syncagent
```

## misc

```bash
kubectl get apibindings
kubectl delete apibindings org.com
# reactivate service in ui

kubectl delete apiresourceschemas.apis.kcp.io --all
kubectl delete apiexport --all
kubectl delete apiexportendpointslices.apis.kcp.io --all
```
