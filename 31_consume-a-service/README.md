# Consume a Service

In this lab you will learn how to consume a service.

## Create an instance of MyService in the consumer workspace

```bash
# switch to the consumer workspace
kubectx consumer@kcp

# note that the APIResouce MyService already exists
kubectl api-resources | grep myservice

# verify you have permission to create an instance of MyService
kubectl auth can-i create MyService

# create the instance of the CRD MyService
kubectl apply -f /training/31_consume-a-service/mymyservice.yaml

# verify
kubectl get myservices.myorg.com
```
