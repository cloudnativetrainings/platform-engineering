# Verify

> IMPORTANT: You are now the service provider!

In this lab you will learn how to verify from the service provider's point of view.

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
# stop kcp locally via <CTRL><C> in the bash running kcp, you do not need it anymore
```
