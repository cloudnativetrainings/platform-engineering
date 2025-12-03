# Sharing Resources

In this lab you will learn how to share CRDs and how CRs will get distrubted across workspaces.

## Providing Resources

```bash
# create a workspace named `provider`
kubectl create-workspace provider --enter
kubectl ws tree

# create the apiresourceschema
kubectl apply -f /training/04_sharing-apis/api/provider/myapiresourceschema.yaml
kubectl get apiresourceschema

# create the apiexport
kubectl apply -f /training/04_sharing-apis/api/provider/myapiexport.yaml
kubectl get apiexport
```

## Consuming Resources

```bash

# create a workspace named `consumer`
kubectl ws :root
kubectl create-workspace consumer --enter
kubectl ws tree

# bind the apiexport of the workspace named `provider` into the workspace named `consumer`
kubectl kcp bind apiexport root:provider:myapiexport --name myapibinding

# verify the resource type named `MySharedResource` exists
kubectl get mysharedresources

# note no CRD was created
kubectl get crds

# note the resource type apears in the api-resources
kubectl api-resources | grep MySharedResource

# apply your first instance of MySharedResource
kubectl apply -f /training/04_sharing-apis/api/consumer/mysharedresource.yaml

# verify your first instance of your shared resource got created
kubectl get mysharedresources
```

## Reading the Resource from the Providers Point of View

```bash
# switch to the workspace named `provider`
kubectl ws :root:provider

# store the url of the apiexport into an env var
export API_ENDPOINT_URL=$(kubectl get apiexportendpointslices.apis.kcp.io myapiexport -o jsonpath={.status.endpoints[0].url})

# verify
echo $API_ENDPOINT_URL 

# get all the api-resources of the workspace named `consumer`
kubectl -s "$API_ENDPOINT_URL/clusters/*" api-resources

# get all mysharedresource instances in the workspace named `consumer`
kubectl -s "$API_ENDPOINT_URL/clusters/*" get mysharedresources -A
```

## Clean-up

```bash
# stop kcp locally via <CTRL><C> in the bash running kcp, we do not need it anymore
```
