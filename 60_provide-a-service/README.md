# Expose a Service

In this lab you will expose a service via KDP.

## Preparations in the KDP Dashboard

Do the following steps in the KDP Dashboard:

### Download the kcp root kubeconfig

* Download the root kcp kubeconfig with KDP Dashboard
* Drag and drop the kubeconfig into your `/training/.secrets/` directory of your codespace.
* Rename the kubeconfig to `kubeconfig-kdp-root.yaml`.

```bash
# due to the training environment you have to add a parameter to the file `/training/.secrets/kubeconfig-kdp-root.yaml`
yq e ".users[0].user.exec.args += \"--oidc-redirect-url=https://$CODESPACE_NAME-8000.app.github.dev\"" -i /training/.secrets/kubeconfig-kdp-root.yaml
```

### Create your service to get provided and consumedd

* add an organization named "provider"
* create a service
  * servicename "myservice"
  * api group "myorg.com"
  * api-syncagent-kubeconfig name "provider-kubeconfig"
  * api-syncagent-kubeconfig namespace named "default"
* activate service named "myservice" in ui
* download the kubeconfig of the service
* drag and drop the kubeconfig into the folder `/training/.secrets/` in your codespace and rename the kubeconfig to `kubeconfig-kdp-provider.yaml`.

## Add Syncagent in provider cluster

```bash
# switch to the provider cluster
kubectx admin@k8s-provider

# create the CRD (note it may be still there from the kcp labs)
kubectl apply -f /training/60_provide-a-service/myservice_crd.yaml

# verify
kubectl get crd myservices.myorg.com

# create the secret holding the kubeconfig to be used by the syncagent
kubectl create secret generic kubeconfig-kcp-provider \
  --namespace default\
  --from-file kubeconfig=/training/.secrets/kubeconfig-kdp-provider.yaml

# allow the syncagent to list objects of myservice
kubectl create clusterrole myservice-syncagent \
  --verb=* \
  --resource=myservices
kubectl create clusterrolebinding myservice-syncagent \
  --clusterrole=myservice-syncagent \
  --serviceaccount=default:myservice-syncagent

# verify permissions of syncagent in provider cluster
kubectl auth can-i list MyService --as=system:serviceaccount:default:myservice-syncagent
kubectl auth can-i watch MyService --as=system:serviceaccount:default:myservice-syncagent

# verify permissions of syncagent in kcp workspace :root:provider
KUBECONFIG=/training/.secrets/kubeconfig-kdp-provider.yaml kubectl auth can-i get logicalclusters/cluster

# get the name of the apiexport, the syncagent configuration has to match the name of the apiexport 
KUBECONFIG=/training/.secrets/kubeconfig-kdp-provider.yaml kubectl get apiexport

# release the syncagent helm chart
helmfile sync -f /training/60_provide-a-service/myservice_syncagent-helmfile.yaml --selector id=myservice-syncagent

# verify pods are running
kubectl get pods

# logs of syncagent
kubectl logs -f -l app.kubernetes.io/instance=myservice-syncagent

# publish resource
kubectl apply -f /training/60_provide-a-service/myservice_published-resource.yaml

# verify the api-resource named `myservice` exists on the provider workspace
KUBECONFIG=/training/.secrets/kubeconfig-kdp-provider.yaml kubectl api-resources

# verify the apiresourceschema got created on the provider workspace
KUBECONFIG=/training/.secrets/kubeconfig-kdp-provider.yaml kubectl get apiresourceschema

# verify the apiexport got created on the provider workspace
KUBECONFIG=/training/.secrets/kubeconfig-kdp-provider.yaml kubectl get apiexport

# add a label on the apiresourceschema
# note this is a known issue which will be fixed soon
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl ws tree
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl ws :root:provider
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl get apiresourceschemas
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl label apiresourceschemas v275e6013.myservices.myorg.com syncagent.kcp.io/api-group=myorg.com

# verify kdp service exists
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl get service.core.kdp.k8c.io
```
