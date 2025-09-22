
# Expose a Service

## Create the Service in the Platform

```bash
# create org named "myorg" in ui

# create service in ui
# servicename "myservice"
# api "myorg.com"
# api-syncagent-kubeconfig name "myorg-myservice-kubeconfig"
# api-syncagent-kubeconfig namespace named "default"

# activate service named "myservice" in ui

# check if the apiexport has been created
# download the syncagent kubeconfig ./secrets/kubeconfig-kcp-myorg-com-myservice.yaml
kubectl --kubeconfig=./.secrets/kubeconfig-kcp-myorg-com-myservice.yaml get apiexport
```

## Create the Service in the Service Cluster

```bash

# add syncagent

kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml create namespace kdp-system

# create the kubeconfig secret
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml \
  create secret generic kubeconfig-kcp-myorg-com-myservice \
  --namespace kdp-system \
  --from-file kubeconfig=./.secrets/kubeconfig-kcp-myorg-com-myservice.yaml

# create the syncagent
helm --kubeconfig=/Users/hubert/git/cloudnativetrainings_platform-engineering/.secrets/kubeconfig-service-cluster.yaml -n kdp-system uninstall myservice-syncagent
helmfile --kubeconfig=/Users/hubert/git/cloudnativetrainings_platform-engineering/.secrets/kubeconfig-service-cluster.yaml sync -f ./service-cluster/myservice_syncagent-helmfile.yaml  --selector id=myservice-syncagent
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml -n kdp-system get pods

# allow syncagent to read the crs
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml apply -f ./service-cluster/myservice_syncagent-rbac.yaml

# verify
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml auth can-i get myservice --as=system:serviceaccount:kdp-system:myservice-syncagent

# publish resource
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml apply -f ./service-cluster/myservice_published-resource.yaml

# check the logs of the syncagent
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml -n kdp-system logs -f -l app.kubernetes.io/name=kcp-api-syncagent

# may restart syncagents
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml -n kdp-system delete pods --all
```

## Fix ApiResourceSchema label issue

```bash
# download kcp admin kubeconfig named "./.secrets/kubeconfig-kcp-admin.yaml"

kubectl --kubeconfig=./.secrets/kubeconfig-kcp-admin.yaml get workspaces
export KUBECONFIG=./.secrets/kubeconfig-kcp-admin.yaml
kubectl ws .
kubectl ws tree
kubectl ws myorg

# verify apiexport
kubectl get apiexport

# verify apibinding
kubectl get apibinding myorg.com -o yaml

# verify apiresourceschema
kubectl get apiresourceschema

# label the apiresourceschema
kubectl label apiresourceschema v275e6013.myservices.myorg.com syncagent.kcp.io/api-group=myorg.com

# # may restart syncagents
# kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml -n kdp-system delete pods --all

# # check the logs of the syncagent
# kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml -n kdp-system logs -f -l app.kubernetes.io/name=kcp-api-syncagent

```

## Create a MyService Object in UI

```bash
# name myservice-01
# namespace default
# message ...

# verify CR was created
kubectl get myservice
```
