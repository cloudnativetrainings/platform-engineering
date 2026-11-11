# Sharing APIs

> IMPORTANT: You are now the service provider!

In this lab you will learn how to share apis.

```bash
# create a workspace named `provider`
kubectl create-workspace provider --enter
```

```bash
# verify
kubectl ws tree
```

```bash
# create the apiresourceschema
kubectl apply -f /training/04_PROVIDER_sharing-apis/myapiresourceschema.yaml
kubectl get apiresourceschema
```

```bash
# create the apiexport
kubectl apply -f /training/04_PROVIDER_sharing-apis/myapiexport.yaml
kubectl get apiexport
```
