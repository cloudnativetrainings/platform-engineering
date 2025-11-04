
# Create Provider Cluster

In this lab you will setup up the provider cluster, which will provide services.

```bash
# create the cluster
make -C /training/11_create-provider-cluster create-cluster

# verify your kubeconfig
kubectx

# verify provider cluster is working
kubectl get nodes
```
