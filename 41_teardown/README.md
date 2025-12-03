# Teardown

In this lab you will cleanup resources for being able to provide services via kdp afterwards.

## Delete Resources in Consumer Workspace

```bash
# connect to kcp via root access
kubectx root@kcp

# switch to the workspace of the consumer
kubectl ws :root:consumer

# delete your instance of MyService
kubectl delete myservice --all

# delete the apibinding
kubectl delete apibinding myapiexport
```

## Delete Resources in Provider Cluster

```bash
# switch to the provider cluster
kubectx admin@k8s-provider 

# delete published resource
kubectl delete publishedresource myservice 

# delete the syncagent
helm uninstall myservice-syncagent

# delete the secret holding the kubeconfig to be used by the syncagent
kubectl delete secret kubeconfig-kcp-provider  

# delete permissions
kubectl delete clusterrole myservice-syncagent
kubectl delete clusterrolebinding myservice-syncagent
```

## Delete Resources in Provider Workspace

```bash
# connect to kcp via root access
kubectx root@kcp

# switch to the workspace of the provider
kubectl ws :root:provider

# delete the apiexport
kubectl delete apiexport myapiexport

# delete the apiresourceschema
kubectl delete apiresourceschema --all
```

## Delete Workspaces

```bash
# switch to the root workspace
kubectl ws :root

# delete the consumer workspace
kubectl delete workspace consumer

# delete the provider workspace
kubectl delete workspace provider
```
