
# Consume a Service

In this lab you will consume a service via KDP.

## Create an instance of MyService

Do the following steps in the KDP Dashboard:

* add an organization named "consumer"
* click the button with the text "external service" and choose the service named "myservice"
* create an instance of the service named "myservice" and fill in a proper name and message

## Verify in provider cluster

```bash
# verify that the instances got synced into the provider cluster
kubectl get myservices.myorg.com -A
```

## Verify in consumer workspace

```bash
# switch to the consumer workspace via admin
KUBECONFIG=/training/.secrets/kdp-root.yaml kubectl ws :root:consumer

# verify myservice is contained in the api-resources
KUBECONFIG=/training/.secrets/kdp-root.yaml kubectl api-resources | grep myservice

# verify there is a binding to the apiexport provided by the :root:provider workspace
KUBECONFIG=/training/.secrets/kdp-root.yaml kubectl get apibinding myorg.com

# verify the instance you created via UI exists in the consumer workspace
KUBECONFIG=/training/.secrets/kdp-root.yaml kubectl get myservice
```
