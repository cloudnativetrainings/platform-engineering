# Provide a Service

In this lab you will learn how to expose a service to consumers.

```bash
# switch to the provider cluster
kubectx admin@k8s-provider
```

## Create the CRD you want to expose

```bash
# create the CRD
kubectl apply -f /training/21_provide-a-service/myservice_crd.yaml

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

# install syncagent
# TODO yaml looks weired
helmfile sync -f /training/21_provide-a-service/myservice_syncagent-helmfile.yaml --selector id=myservice-syncagent

# verify logs of syncagent pods
kubectl logs -f --prefix -l app.kubernetes.io/instance=myservice-syncagent

# allow the syncagent to list objects of myservice
kubectl create clusterrole myservice-syncagent \
  --verb=* \
  --resource=myservices
kubectl create clusterrolebinding myservice-syncagent \
  --clusterrole=myservice-syncagent \
  --serviceaccount=default:myservice-syncagent

# verify
kubectl auth can-i list MyService --as=system:serviceaccount:default:myservice-syncagent
kubectl auth can-i watch MyService --as=system:serviceaccount:default:myservice-syncagent

# publish a resource
kubectl apply -f /training/21_provide-a-service/myservice_published-resource.yaml

# logs of syncagent pods
kubectl logs -f -l app.kubernetes.io/instance=myservice-syncagent

# for sure there is no instance of myservice, yet
kubectl get myservices.myorg.com -A
```
