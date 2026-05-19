
# Verify

In this lab you will learn how to verify that KDP created the instance in the provider cluster.

## Verify in provider cluster

> IMPORTANT: You are now the service provider!

The user of the UI (e.g., a dev needing some infra) created some infra instance. Now, let's verify if the instance pops up in the provider cluster. If that is the case, some operator running in the provider cluster will recognize this and can create the actual infra in the provider cluster. This is out of scope for this workshop.

```bash
# switch to the provider cluster
kubectx admin@k8s-provider

# verify that the instances got synced into the provider cluster
kubectl get myservices.myorg.com -A

# verify that the instance contains the message you have chosen previously
kubectl get myservices.myorg.com -A -o yaml
```

## Verify in consumer workspace

> IMPORTANT: You are now the platform administrator!

```bash
# switch to the consumer workspace via admin
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl ws :root:consumer

# verify myservice is contained in the api-resources
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl api-resources | grep myservices

# verify there is a binding to the apiexport provided by the :root:provider workspace
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl get apibinding myorg.com

# verify the instance you created via UI exists in the consumer workspace
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl get myservice
```
