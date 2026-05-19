# Provide a Service

> IMPORTANT: You are now the service provider!

In this lab you will learn how to provide a service.

## Create the APIExport in the provider workspace

```bash
# connect to kcp via provider access
kubectx provider@kcp

# create the apiexport
kubectl apply -f /training/21_PROVIDER_provide-a-service/myapiexport.yaml

# verify
kubectl get apiexport myapiexport

# get the apiresourceschema
# note this does not exist yet — the syncagent (installed later) will generate it from the CRD you will create in the next section
kubectl get apiresourceschema
```

## Create the CRD you want to expose

```bash
# switch to the provider cluster
kubectx admin@k8s-provider

# create the CRD
kubectl apply -f /training/21_PROVIDER_provide-a-service/myservice_crd.yaml

# verify
kubectl get crd myservices.myorg.com
```

## Install the SyncAgent for the Provider

```bash
# create the secret the syncagent needs for talking to the workspace provider
kubectl create secret generic kubeconfig-kcp-provider \
  --from-file kubeconfig=/training/.secrets/kubeconfig-kcp-provider-flat.yaml

# verify
kubectl get secrets kubeconfig-kcp-provider -o jsonpath="{.data.kubeconfig}" | base64 -d

# release the syncagent helm chart
helmfile sync -f /training/21_PROVIDER_provide-a-service/myservice_syncagent-helmfile.yaml --selector id=myservice-syncagent

# verify logs of syncagent pods
kubectl logs -f --prefix -l app.kubernetes.io/instance=myservice-syncagent

# allow the syncagent to list objects of myservice
kubectl create clusterrole myservice-syncagent \
  --verb='*' \
  --resource=myservices
kubectl create clusterrolebinding myservice-syncagent \
  --clusterrole=myservice-syncagent \
  --serviceaccount=default:myservice-syncagent-api-syncagent

# verify
kubectl auth can-i list MyService --as=system:serviceaccount:default:myservice-syncagent-api-syncagent
kubectl auth can-i watch MyService --as=system:serviceaccount:default:myservice-syncagent-api-syncagent

# publish a resource
kubectl apply -f /training/21_PROVIDER_provide-a-service/myservice_published-resource.yaml

# logs of syncagent pods
kubectl logs -f -l app.kubernetes.io/instance=myservice-syncagent

# for sure there is no instance of myservice, yet
kubectl get myservices.myorg.com -A

# verify if syncagents created the apiresourceschema in the provider workspace
kubectx provider@kcp
kubectl get apiresourceschema
```
