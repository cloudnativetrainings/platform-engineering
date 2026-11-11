# Consuming APIs

> IMPORTANT: You are now the service consumer!

In this lab you will learn how to consume APIs.

```bash
# create a workspace named `consumer`
kubectl ws :root
kubectl create-workspace consumer --enter
kubectl ws tree
```

```bash
# bind the apiexport of the workspace named `provider` into the workspace named `consumer`
kubectl apply -f /training/05_CONSUMER_consuming-apis/myapibinding.yaml
```

```bash
# note no CRD was created
kubectl get crds
```

```bash
# note the resource type appears in the api-resources
kubectl api-resources | grep MySharedResource
```

```bash
# apply your first instance of MySharedResource
kubectl apply -f /training/05_CONSUMER_consuming-apis/mysharedresource.yaml
```

```bash
# verify your first instance of your shared resource got created
kubectl get mysharedresources
```
