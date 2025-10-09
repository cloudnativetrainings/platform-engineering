
## providing stuff

```bash

kubectx kcp-root
kubectl ws .
kubectl ws tree
kubectl get pods

# verify
kubectl api-resources

# does not work
kubectl run -it nginx --image nginx


kubectl create-workspace service-provider --enter
kubectl ws tree

# TODO path
kubectl apply -f /training/02/myapiexport.yaml
kubectl get apiexport

kubectl apply -f /training/02/myapiresourceschema.yaml
kubectl get apiresourceschema

# TODO understand
# `APIResourceSchema` instance does not have a backing API server, and instead it simply describes an API that we can pass around and refer to. By decoupling the schema definition from serving, API owners can be more explicit about API evolution.
```

# create kubeconfig for service-provider

```bash
# TODO is ws service-provider right with kcp-admin kubeconfig?
kubectl create sa --namespace default service-provider
kubectl apply -f satoken.yaml
kubectl get secret service-provider-secret -o yaml
kubectl get secret service-provider-secret -o jsonpath={.data.token} | base64 -d > /training/.secrets/kcp-service-provider-token.yaml


kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-service-provider.yaml config set-cluster kcp-service-provider --server https://internal.$DOMAIN:8443/clusters/root:service-provider --certificate-authority=/training/.secrets/kcp-front-proxy-cert-ca.crt


kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-service-provider.yaml config set-credentials kcp-service-provider --token=/training/.secrets/kcp-service-provider-token.yaml
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-service-provider.yaml config set-context kcp-service-provider --cluster=kcp-service-provider --user=kcp-service-provider
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-service-provider.yaml config use-context kcp-service-provider

KUBECONFIG="/training/.secrets/kubeconfig-platform-cluster.yaml:/training/.secrets/kubeconfig-service-cluster.yaml:/training/.secrets/kubeconfig-kcp-admin.yaml:/training/.secrets/kubeconfig-kcp-service-provider.yaml" kubectl config view --raw > /training/.secrets/kubeconfig.yaml

# verify
kubectx kcp-service-provider

kubectl ws tree
# does not work due to the user has no permission yet

# TODO name of context should be kcp-admin
kubectx kcp-root
kubectl ws :root:service-provider


kubectl create clusterrolebinding service-provider --clusterrole=cluster-admin --serviceaccount=default:service-provider

# WTF
kubectl get clusterrolebindings.rbac.authorization.k8s.io 
NAME               ROLE                        AGE
service-provider   ClusterRole/cluster-admin   14s
workspace-admin    ClusterRole/cluster-admin   23h

# verify
kubectx kcp-service-provider

```
