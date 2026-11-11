# Create Platform Cluster

> IMPORTANT: You are now the platform administrator!

In this lab you will set up the platform cluster, which will run kcp.

## Create Kubernetes Cluster

```bash
# verify kubeone got installed
kubeone version
```

```bash
# create the cluster
make -C /training/10_ADMIN_create-platform-cluster create-cluster
```

```bash
# verify your kubeconfig
kubectx
```

```bash
# verify platform cluster is working
kubectl get nodes
```
