# Sharing APIs

> IMPORTANT: You are now the service provider!

In this lab you will learn how to share apis.

```bash
# create a workspace named `provider`
kubectl create-workspace provider --enter

# verify
kubectl ws tree

# create the apiresourceschema
kubectl apply -f /training/04_PROVIDER_sharing-apis/myapiresourceschema.yaml
kubectl get apiresourceschema

# create the apiexport
kubectl apply -f /training/04_PROVIDER_sharing-apis/myapiexport.yaml
kubectl get apiexport
```
