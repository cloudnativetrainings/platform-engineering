# Prepare Consumer Workspace

> IMPORTANT: You are now the platform administrator!

In this lab you prepare the workspace for the consumer.

```bash
# connect to kcp via root access
kubectx root@kcp

# switch to the workspace of the consumer
kubectl ws :root:consumer

# note that the resource type myservice does not exist yet
kubectl api-resources | grep myservice

# bind the apiexport from the provider workspace into the consumer workspace
kubectl apply -f /training/30_ADMIN_prepare-consumer-workspace/myapibinding.yaml

# now the resource type has to exist
kubectl api-resources | grep myservice

# give permissions to consumer
kubectl create clusterrole myservice-consumer \
  --verb='*' \
  --resource=myservices
kubectl create clusterrolebinding myservice-consumer \
  --clusterrole=myservice-consumer \
  --serviceaccount=default:consumer

# verify
kubectl auth can-i create MyService --as=system:serviceaccount:default:consumer
kubectl auth can-i list MyService --as=system:serviceaccount:default:consumer
```
