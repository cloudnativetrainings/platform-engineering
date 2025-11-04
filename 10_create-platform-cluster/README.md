# Create Platform Cluster

In this lab you will setup up the platform cluster, which will run kcp.

## Create Kubernetes Cluster

```bash
# create the cluster
make -C /training/10_create-platform-cluster create-cluster

# verify your kubeconfig
kubectx

# verify platform cluster is working
kubectl get nodes
```
