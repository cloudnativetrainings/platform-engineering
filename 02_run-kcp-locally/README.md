
# Run kcp locally

In this lab you will run kcp locally and you will see the differences from a "real" Kubernetes cluster.

## Start kcp

```bash
# [BASH2] start kcp
kcp start

# verify
ps aux | grep kcp

# explore and explain directory `/training/.kcp/`
tree /training/.kcp
```

## Explore the "Kubernetes Cluster"

```bash
# set the kubeconfig
cp /training/.kcp/admin.kubeconfig /training/.secrets/kubeconfig.yaml

# verify
kubectl api-resources

# does not work
kubectl run -it nginx --image nginx
```
