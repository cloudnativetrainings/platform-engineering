
# Expose a Service

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-a-long-lived-api-token-for-a-serviceaccount

https://docs.kcp.io/api-syncagent/v0.4/getting-started/

apibinding vorher anlegen sonst startet syncagent

servicecluster crd
published resource

apibinding muss im workspace existieren sonst startet der syncagent auf service cluster

fürs erstellen von apibinding bietet kcp plugin kubectl kcp bind apiexport

demo env verwendet für syncagent certs, nicht token, kann sein das es noch probleme gibt, bei problemen -> daniel

## Create the Service in the Service Cluster

```bash

# switch to service cluster
kubectx service-cluster

# add syncagent
# ! on the service cluster
kubectl create namespace kdp-system

# create the kubeconfig secret
# ! on the service cluster
kubectl create secret generic kubeconfig-kcp-myorg-com-myservice \
  --namespace kdp-system \
  --from-file kubeconfig=/training/.secrets/myorg.com-kubeconfig

# create the syncagent
# ! on the service cluster
# helm -n kdp-system uninstall myservice-syncagent
helmfile sync -f /training/service-cluster/myservice/myservice/myservice_syncagent-helmfile.yaml  --selector id=myservice-syncagent
kubectl -n kdp-system get pods

# allow syncagent to read the crs
# ! on the service cluster
kubectl apply -f /training/service-cluster/myservice/myservice_syncagent-rbac.yaml

# verify
# ! on the service cluster
kubectl auth can-i get myservice --as=system:serviceaccount:kdp-system:myservice-syncagent

# publish resource
# ! on the service cluster
kubectl apply -f /training/service-cluster/myservice/myservice_published-resource.yaml

# check the logs of the syncagent
# ! on the service cluster
kubectl -n kdp-system logs -f -l app.kubernetes.io/name=kcp-api-syncagent

# may restart syncagents
# ! on the service cluster
kubectl -n kdp-system delete pods --all
```

## Fix ApiResourceSchema label issue

```bash

# download kcp admin kubeconfig
# drag and drop the kubeconfig into "/training/.secrets/kubeconfig-kcp-admin.yaml" in your codespace

# merge the kubeconfigs into one
KUBECONFIG=/training/.secrets/kubeconfig-platform-cluster.yaml:/training/.secrets/kubeconfig-service-cluster.yaml:/training/.secrets/myorg.com-kubeconfig:/training/.secrets/kubeconfig-kcp-admin.yaml kubectl config view --raw > /training/.secrets/kubeconfig.yaml

# switch to kcp-admin cluster
# TODO name of context
kubectx root

# add it to dex redirecturis
kubectx pla
helmfile sync --file /training/platform-cluster/helm/helmfile.yaml --selector id=dex
kubectx root
# add #TODO '--oidc-redirect-url=https://humble-space-guide-gx66rgjjx7qfvrw6-8000.app.github.dev' to kubeconfig

# ! on the kcp-admin cluster
kubectl get workspaces

# ! on the kcp-admin cluster
kubectl ws .
kubectl ws tree
kubectl ws myorg

# verify apiexport
# ! on the kcp-admin cluster
kubectl get apiexport

# verify apibinding
# ! on the kcp-admin cluster
kubectl get apibinding myorg.com -o yaml

# verify apiresourceschema
# ! on the kcp-admin cluster
kubectl get apiresourceschema

# label the apiresourceschema
# ! on the kcp-admin cluster
kubectl label apiresourceschema v275e6013.myservices.myorg.com syncagent.kcp.io/api-group=myorg.com

# may restart syncagents
# ! on service cluster
kubectx service-cluster
kubectl -n kdp-system delete pods --all

# # check logs of the syncagent
# ! on service cluster
kubectl -n kdp-system logs -f -l app.kubernetes.io/name=kcp-api-syncagent
```
