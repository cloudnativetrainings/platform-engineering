
# Workspaces

In this lab you will learn about multitenancy features in kcp.

```bash
# see your current workspace
kubectl ws .

# see the workspace tree
kubectl ws tree

# create workspaces
kubectl create-workspace my-workspace-1
kubectl create-workspace my-workspace-2

# verify
kubectl ws tree
kubectl get ws

# switch to the workspace named `my-workspace-1`
kubectl ws :root:my-workspace-1
kubectl ws tree
kubectl get ws

# switch to the root workspace again
kubectl ws :root
kubectl ws tree
kubectl get ws

# see the workspacetypes
kubectl get workspacetypes

# see all kcp related api-resources
kubectl api-resources | grep kcp.io
```

## Multitenancy

```bash
# create a configmap in the workspace named `my-workspace-1`
kubectl ws :root:my-workspace-1
kubectl ws .
kubectl ws tree
kubectl create configmap my-configmap --from-literal=key="some value"
kubectl get cm
kubectl get cm my-configmap -o jsonpath="{.data.key}"

# create a configmap in the workspace named `my-workspace-2`
kubectl ws :root:my-workspace-2
kubectl ws .
kubectl ws tree
kubectl create configmap my-configmap --from-literal=key="another value"
kubectl get cm
kubectl get cm my-configmap -o jsonpath="{.data.key}"
```

## Verify data in etcd

Lets take a look at the data stored in etcd.

```bash
# verify if none of the above created configmaps are readable at the root workspace
kubectl ws :root
kubectl ws .
kubectl ws tree
kubectl get cm

# get existing workspaces via etcdctl
etcdctl get --prefix /registry/tenancy.kcp.io/workspaces/ --keys-only 

# get the logical-cluster-name of the workspace named `my-workspace-1`
kubectl get ws my-workspace-1 -o jsonpath='{.metadata.annotations.internal\.tenancy\.kcp\.io/cluster}'

# query etcd for the configmap named `my-configmap` in the workspace named `my-workspace-1` in the namespace named `default`
etcdctl get /registry/core/configmaps/<FILL-IN-THE-LOGICAL-CLUSTER-NAME>/default/my-configmap

# query etcd for all configmaps in the workspace named `my-workspace-1` in the namespace named `default`
etcdctl get --prefix /registry/core/configmaps/<FILL-IN-THE-LOGICAL-CLUSTER-NAME>/default/ --keys-only 
```

## The `home` workspace

Something, you should be aware of.

```bash
# switch to the workspace named `my-workspace-2`
kubectl ws :root:my-workspace-2
kubectl ws .
kubectl ws tree

# switch to your home workspace, note this one has a very cryptic name
kubectl ws 
kubectl ws .
kubectl ws tree

# try to switch to the root workspace, note that you will be stuck in your home workspace
kubectl ws :
kubectl ws .
kubectl ws tree

# finally, switch to the root workspace
kubectl ws :root
```

## Clean-up

```bash
# delete all the workspaces
kubectl delete ws --all
```
