
# Run kcp locally

In this lab you will run kcp locally and you will see the differences to a "real" Kubernetes Cluster.

## Start kcp

```bash
# start kcp in a seperate bash
kcp start

# verify in a new bash
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
