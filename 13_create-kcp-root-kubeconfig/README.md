
# Create the KCP Root Kubeconfig

In this lab you will create the kubeconfig for being able to do admin tasks on kcp running inside the platform cluster.

Documentation can be found in the [kcp-dev github repository](https://github.com/kcp-dev/helm-charts/tree/main/charts/kcp#initial-access).

```bash
# switch to the platform cluster
kubectx admin@k8s-platform

# switch to the kcp-system namespace
kubens kcp-system
```

## Preps

```bash
# obtain the CA cert of the kcp-front-proxy
kubectl get secret kcp-front-proxy-cert -o=jsonpath='{.data.tls\.crt}' | base64 -d > /training/.secrets/kcp-front-proxy-cert-ca.crt

# create cluster admin cert
kubectl apply -f /training/13_create-kcp-root-kubeconfig/cluster-admin-client-cert.yaml

# verify
kubectl get cert cluster-admin-client-cert
kubectl get secret cluster-admin-client-cert

# get data from cluster admin cert
kubectl get secret cluster-admin-client-cert -o=jsonpath='{.data.tls\.crt}' | base64 -d > /training/.secrets/cluster-admin-client-cert.crt
kubectl get secret cluster-admin-client-cert -o=jsonpath='{.data.tls\.key}' | base64 -d > /training/.secrets/cluster-admin-client-cert.key
```

## Create the kubeconfig

```bash
# create cluster section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-root.yaml config set-cluster kcp-base --server https://internal.$PLATFORM_DOMAIN:8443 --certificate-authority=/training/.secrets/kcp-front-proxy-cert-ca.crt
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-root.yaml config set-cluster kcp-root --server https://internal.$PLATFORM_DOMAIN:8443/clusters/root --certificate-authority=/training/.secrets/kcp-front-proxy-cert-ca.crt

# create user section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-root.yaml config set-credentials root@kcp --client-certificate=/training/.secrets/cluster-admin-client-cert.crt --client-key=/training/.secrets/cluster-admin-client-cert.key

# create context section in kubeconfig
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-root.yaml config set-context base@kcp --cluster=kcp-base --user=root@kcp
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-root.yaml config set-context root@kcp --cluster=kcp-root --user=root@kcp
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-root.yaml config use-context root@kcp

# squash the kubeconfigs
make -C /training squash-kubeconfigs

# verify
kubectx
kubectl ws tree
```
