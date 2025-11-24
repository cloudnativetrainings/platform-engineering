# Expose a Service

In this lab you will expose a service via KDP.

## Preparations in the KDP Dashboard

Do the following steps in the KDP Dashboard:

### Download the kcp root kubeconfig

* Download the root kcp kubeconfig with KDP Dashboard
* Drag and drop the kubeconfig into your `/training/.secrets/` directory of your codespace.
* Rename the kubeconfig to `kubeconfig-kcp-root.yaml`.

```bash
# due to the training environment you have to add a parameter to the file `/training/.secrets/kubeconfig-kcp-root.yaml`
yq e ".users[0].user.exec.args += \"--oidc-redirect-url=https://$CODESPACE_NAME-8000.app.github.dev\"" -i /training/.secrets/kubeconfig-kcp-root.yaml
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
* drag and drop the kubeconfig into the folder `/training/.secrets/` in your codespace

## Syncagent in provider cluster

```bash
# switch to the provider cluster
kubectx admin@k8s-provider

# create the CRD
kubectl apply -f /training/60_provide-a-service/myservice_crd.yaml

# verify
kubectl get crd myservices.myorg.com

# create the secret holding the kubeconfig to be used by the syncagent
kubectl create secret generic kubeconfig-kcp-provider \
  --namespace default\
  --from-file kubeconfig=/training/.secrets/myorg.com-kubeconfig

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
# TODO does not work yet
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl auth can-i get logicalclusters/cluster
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl auth can-i get apiexport/myapiexport
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl auth can-i create apiexport
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl auth can-i list MyService 
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl auth can-i watch MyService 

# get the name of the apiexport, the syncagent configuration has to match the name of the apiexport 
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl get apiexport

# install syncagent
# TODO yaml looks weired
helmfile sync -f /training/60_provide-a-service/myservice_syncagent-helmfile.yaml --selector id=myservice-syncagent

# verify pods are running
kubectl get pods

# logs of syncagent
kubectl logs -f -l app.kubernetes.io/instance=myservice-syncagent

# publish resource
kubectl apply -f /training/60_provide-a-service/myservice_published-resource.yaml

# verify the api-resource named `myservice` exists on the provider workspace
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl api-resources

# verify the apiresourceschema got created on the provider workspace
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl get apiresourceschema

# verify the apiexport got created on the provider workspace
KUBECONFIG=/training/.secrets/myorg.com-kubeconfig kubectl get apiexport

# add a label on the apiresourceschema
# note this is a known issue which will be fixed soon
KUBECONFIG=/training/.secrets/kubeconfig-kcp-root.yaml kubectl ws tree
KUBECONFIG=/training/.secrets/kubeconfig-kcp-root.yaml kubectl ws :root:provider
KUBECONFIG=/training/.secrets/kubeconfig-kcp-root.yaml kubectl get apiresourceschemas
KUBECONFIG=/training/.secrets/kubeconfig-kcp-root.yaml kubectl label apiresourceschemas v275e6013.myservices.myorg.com syncagent.kcp.io/api-group=myorg.com

# verify kdp service exists
KUBECONFIG=/training/.secrets/kubeconfig-kcp-root.yaml kubectl get service.core.kdp.k8c.io
```
