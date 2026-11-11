# Consume a Service

> IMPORTANT: You are now the service consumer!

In this lab you will learn how to consume a service.

## Create an instance of MyService in the consumer workspace

```bash
# switch to the consumer workspace
kubectx consumer@kcp
```

```bash
# note that the APIResource MyService already exists
kubectl api-resources | grep myservice
```

```bash
# verify you have permission to create an instance of MyService
kubectl auth can-i create MyService
```

```bash
# create the instance of the CRD MyService
kubectl apply -f /training/31_CONSUMER_consume-a-service/mymyservice.yaml
```

```bash
# verify
kubectl get myservices.myorg.com
```
