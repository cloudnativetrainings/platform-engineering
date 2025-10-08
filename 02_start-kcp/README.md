
## Start kcp

```bash

# start kcp in a seperate bash
kcp start

# verify
ps aux | grep kcp

# TODO explore and explain .kcp dir

```

## Explore k8s cluster

```bash
# set the kubeconfig
# TODO integrate in other kubeconfig way of doing things?
export KUBECONFIG=/training/.kcp/admin.kubeconfig

# verify
kubectl api-resources

# does not work
kubectl run -it nginx --image nginx
```

## Workspaces

```bash

kubectl ws .

kubectl ws tree

kubectl create-workspace my-workspace-1

kubectl create-workspace my-workspace-2

kubectl ws tree

# TODO del ws

# TODO workspacetypes kubectl get workspacetypes
# https://docs.kcp.io/kcp/v0.11/concepts/quickstart-tenancy-and-apis/#understand-workspace-types

kubectl get ws

# TODO  wokspace navigation
# TODO https://kubermatic.slack.com/archives/C041GAPBM0D/p1759748894315249
# TODO PR for enhancing comment? "kubectl ws :root" does the trick, not "kubectl ws :"
  # kubectl ws
  # Current workspace is 'kvdk2spgmbix'.
  # Note: 'kubectl ws' now matches 'cd' semantics: go to home workspace. 'kubectl ws -' to go back. 'kubectl ws .' to print current workspace.



```

## multitenancy

```bash
# workspace 1
kubectl ws :root:my-workspace-1
kubectl ws .
kubectl ws tree
kubectl create configmap my-configmap --from-literal=key="some value"
kubectl get cm
kubectl get cm my-configmap -o jsonpath="{.data.key}"

# workspace 2
kubectl ws :root:my-workspace-2
kubectl ws .
kubectl ws tree
kubectl create configmap my-configmap --from-literal=key="another value"
kubectl get cm
kubectl get cm my-configmap -o jsonpath="{.data.key}"



# root
kubectl ws :root
kubectl ws .
kubectl ws tree
kubectl get cm
```
<!-- TODO is provider and consumer really the best naming... ??? -->

## providing stuff

```bash

kubectl create-workspace service-provider --enter
kubectl ws tree

# TODO path
kubectl apply -f /training/02/provider/myapiexport.yaml
kubectl get apiexport

kubectl apply -f /training/02/provider/myapiresourceschema.yaml
kubectl get apiresourceschema

# TODO understand
# `APIResourceSchema` instance does not have a backing API server, and instead it simply describes an API that we can pass around and refer to. By decoupling the schema definition from serving, API owners can be more explicit about API evolution.

```

## consuming stuff

```bash

# consumer
kubectl ws :root
kubectl create-workspace service-consumer --enter
kubectl ws tree

kubectl kcp bind apiexport root:service-provider:myapiexport --name my-binding --accept-permission-claim configmaps.core --insecure-skip-tls-verify

kubectl get crds
kubectl get mysharedresources

# TODO path
kubectl apply -f /training/02/consumer/mysharedresource_01.yaml
kubectl get mysharedresources




```

## provider again

```bash
kubectl ws :root:service-provider

# does not work

kubectl get logicalclusters.core.kcp.io 
https://10.0.10.233:6443/clusters/1hvyg9k40x2jnsgs



kubectl get apiexport myapiexport -o json | jq '.status.virtualWorkspaces[].url'
kubectl -s 'https://192.168.32.7:6443/services/apiexport/1ctnpog1ny8bnud6/cowboys/clusters/*' api-resources
kubectl -s 'https://192.168.32.7:6443/services/apiexport/1ctnpog1ny8bnud6/cowboys/clusters/*' get cowboys -A


```
