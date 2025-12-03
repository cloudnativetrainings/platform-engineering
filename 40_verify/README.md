# Verify

In this lab you will verify that the instance of MyService was created in the provider cluster. Afterwards a Controller in the provider can pick up the work.

## Verify the instance was synced to the provider cluster

```bash
# switch to the provider cluster
kubectx admin@k8s-provider

# verify the instance exists
kubectl get myservices.myorg.com -A
```
