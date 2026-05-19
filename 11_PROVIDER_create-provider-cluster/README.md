
# Create Provider Cluster

> IMPORTANT: You are now the service provider!

In this lab you will set up the provider cluster, which will provide services.

```bash
# create the cluster
make -C /training/11_PROVIDER_create-provider-cluster create-cluster

# verify your kubeconfig
kubectx

# verify provider cluster is working
kubectl get nodes
```
