# Create the KCP Consumer Kubeconfig

In this lab you will create a workspace for the consumer and create the kubeconfig enabling the Consumer to interact with it.

```bash
# switch to kcp via root
kubectx root@kcp

# create the consumer workspace and enter it
kubectl create-workspace consumer --enter
kubectl ws tree
```

## Preps

```bash
# create sa token
kubectl create sa consumer

# create secret for sa token
kubectl apply -f /training/15_create-kcp-consumer-kubeconfig/consumer_service-account-token-secret.yaml

# verify
kubectl get secret consumer-secret

# get the token
kubectl get secret consumer-secret -o jsonpath={.data.token} | base64 -d > /training/.secrets/kcp-consumer-token.yaml
```

## Create the kubeconfig

```bash
# create cluster section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-consumer.yaml config set-cluster kcp-consumer --server https://internal.$PLATFORM_DOMAIN:8443/clusters/root:consumer --certificate-authority=/training/.secrets/kcp-front-proxy-cert-ca.crt 

# create user section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-consumer.yaml config set-credentials consumer@kcp --token=/training/.secrets/kcp-consumer-token.yaml

# create context section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-consumer.yaml config set-context consumer@kcp --cluster=kcp-consumer --user=consumer@kcp
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-consumer.yaml config use-context consumer@kcp

# flatten kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-consumer.yaml config view --flatten > /training/.secrets/kubeconfig-kcp-consumer-flat.yaml
yq e '.users[0].user.token = load_str("/training/.secrets/kcp-consumer-token.yaml")' -i /training/.secrets/kubeconfig-kcp-consumer-flat.yaml

# del the non-flat kubeconfig
rm /training/.secrets/kubeconfig-kcp-consumer.yaml

# squash the kubeconfigs
make -C /training squash-kubeconfigs

# verify
kubectx consumer@kcp
kubectl api-resources
```
