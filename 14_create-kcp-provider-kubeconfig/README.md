# Create the KCP Provider Kubeconfig

In this lab you will create a workspace for the provider and create the kubeconfig enabling the Provider to interact with it.

```bash
# switch to kcp via root
kubectx root@kcp

# create the provider workspace and enter it
kubectl create-workspace provider --enter
kubectl ws tree
```

## Preps

```bash
# create sa token
kubectl create sa provider

# create secret for sa token
kubectl apply -f /training/14_create-kcp-provider-kubeconfig/provider_service-account-token-secret.yaml

# verify
kubectl get secret provider-secret

# get the token
kubectl get secret provider-secret -o jsonpath={.data.token} | base64 -d > /training/.secrets/kcp-provider-token.yaml
```

## Create the kubeconfig

```bash
# create cluster section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-provider.yaml config set-cluster kcp-provider --server https://internal.$PLATFORM_DOMAIN:8443/clusters/root:provider --certificate-authority=/training/.secrets/kcp-front-proxy-cert-ca.crt 

# create user section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-provider.yaml config set-credentials provider@kcp --token=/training/.secrets/kcp-provider-token.yaml

# create context section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-provider.yaml config set-context provider@kcp --cluster=kcp-provider --user=provider@kcp
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-provider.yaml config use-context provider@kcp

# flatten kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-provider.yaml config view --flatten > /training/.secrets/kubeconfig-kcp-provider-flat.yaml
yq e '.users[0].user.token = load_str("/training/.secrets/kcp-provider-token.yaml")' -i /training/.secrets/kubeconfig-kcp-provider-flat.yaml

# del the non-flat kubeconfig
rm /training/.secrets/kubeconfig-kcp-provider.yaml

# squash the kubeconfigs
make -C /training squash-kubeconfigs

# verify
kubectx provider@kcp
kubectl api-resources
```
