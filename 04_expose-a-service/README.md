
# Expose a Service

## Create the Service in the Platform

<!-- TODO domain env var - do i need 2? -->

```bash
# create org named "myorg" in ui

# create service in ui
# servicename "myservice"
# api "myorg.com"
# api-syncagent-kubeconfig name "myorg-myservice-kubeconfig"
# api-syncagent-kubeconfig namespace named "default"

# activate service named "myservice" in ui
# download the kubeconfig of the service
# drag and drop the kubeconfig into the folder `/training/.secrets/` in your codespace

# merge the kubeconfigs into one

# TODO this breaks the naming of the contexts in kubeconfig file
KUBECONFIG=/training/.secrets/kubeconfig-platform-cluster.yaml:/training/.secrets/kubeconfig-service-cluster.yaml:/training/.secrets/myorg.com-kubeconfig kubectl config view --raw > /training/.secrets/kubeconfig.yaml

# rename the kubeconfig contexts
kubectl config rename-context kcp kcp-myservice-myorg-com

# switch to kcp-myservice-myorg-com cluster
kubectx kcp-myservice-myorg-com

# verify your kcp kubeconfig
# ! on the kcp-myservice-myorg-com cluster
kubectl ws .

# check if the apiexport has been created
# ! on the kcp-myservice-myorg-com cluster
kubectl get apiexport
```

<!-- TODO on running syncagent 0.4.0

{"level":"fatal","time":"2025-09-26T12:13:05.531Z","caller":"api-syncagent/main.go:82","msg":"Sync Agent has encountered an error","error":"failed to resolve APIExport/EndpointSlice: failed to resolve APIExportEndpointSlice: failed to get APIExportEndpointSlice \"myorg.com\": apiexportendpointslices.apis.kcp.io \"myorg.com\" is forbidden: User \"kdp:servlet-myorg.com@ezgo3l72yr6rwoht\" cannot get resource \"apiexportendpointslices\" in API group \"apis.kcp.io\" at the cluster scope: access denied"}
 -->

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

# TODO place this somewhere else in lab 02 setup platform
# get the redirect url
echo https://$CODESPACE_NAME-8000.app.github.dev

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

## Create a MyService Object in UI

```bash
# name myservice-01
# namespace default
# message ...

# verify CR was created
# ! on service cluster
kubectl get myservice

# TODO namespace looks wrong
```
