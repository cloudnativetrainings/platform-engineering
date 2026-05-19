# Consuming APIs

> IMPORTANT: You are now the service consumer!

In this lab you will learn how to consume APIs.

```bash
# create a workspace named `consumer`
kubectl ws :root
kubectl create-workspace consumer --enter
kubectl ws tree

# bind the apiexport of the workspace named `provider` into the workspace named `consumer`
kubectl apply -f /training/05_CONSUMER_consuming-apis/myapibinding.yaml

# note no CRD was created
kubectl get crds

# note the resource type appears in the api-resources
kubectl api-resources | grep MySharedResource

# apply your first instance of MySharedResource
kubectl apply -f /training/05_CONSUMER_consuming-apis/mysharedresource.yaml

# verify your first instance of your shared resource got created
kubectl get mysharedresources
```
