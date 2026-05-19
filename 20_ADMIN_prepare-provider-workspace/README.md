# Prepare Provider Workspace

> IMPORTANT: You are now the platform administrator!

In this lab you prepare the workspace for the provider.

## Grant permissions to the Provider ServiceAccount

The syncagent, running in the Kubernetes cluster of the provider, needs permissions in the kcp workspace.

```bash
# connect to kcp via root access
kubectx root@kcp

# switch to the workspace of the provider
kubectl ws :root:provider

# give the provider admin rights in his workspace
kubectl create clusterrolebinding provider:admin \
  --clusterrole=cluster-admin \
  --serviceaccount=default:provider

# verify
kubectl auth can-i get logicalclusters/cluster --as=system:serviceaccount:default:provider
kubectl auth can-i get apiexport --as=system:serviceaccount:default:provider
kubectl auth can-i create apiexport --as=system:serviceaccount:default:provider
```

## Verify the provider has proper permissions

```bash
# connect to kcp via provider access
kubectx provider@kcp

# verify your permissions
kubectl auth can-i get logicalclusters/cluster
kubectl auth can-i get apiexport
kubectl auth can-i create apiexport
```
