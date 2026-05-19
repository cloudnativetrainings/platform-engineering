# Teardown

In this lab you will clean up resources so that you can provide services via KDP afterwards.

## Delete Resources in Consumer Workspace

> IMPORTANT: You are now the service consumer!

```bash
# switch to the workspace of the consumer
kubectx consumer@kcp

# delete your instance of MyService
kubectl delete myservice --all
```

## Delete Provider Resources

> IMPORTANT: You are now the service provider!

### Delete Resources in Provider Kubernetes Cluster

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

# delete the CRD
kubectl delete crd myservices.myorg.com
```

### Delete Resources in Provider Workspace

```bash
# switch to the workspace of the provider
kubectx provider@kcp

# delete the apiexport
kubectl delete apiexport myapiexport

# delete the apiresourceschema
kubectl delete apiresourceschema --all
```

## Delete Other Resources

> IMPORTANT: You are now the platform administrator!

### Delete Consumer Resources

```bash
# connect to kcp via root access
kubectx root@kcp

# switch to the workspace of the consumer
kubectl ws :root:consumer

# delete the apibinding
kubectl delete apibinding myapibinding
```

### Delete the Workspaces

```bash
# switch to the root workspace
kubectl ws :root

# delete the consumer workspace
kubectl delete workspace consumer

# delete the provider workspace
kubectl delete workspace provider
```

> NOTE: If deletion of workspaces takes too long, you can speed things up by deleting the finalizers, e.g. via `kubectl edit workspace provider`. DO NOT DO THIS IN PROD!
