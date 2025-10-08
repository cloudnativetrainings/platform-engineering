
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
