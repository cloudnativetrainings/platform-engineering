
```bash

kubectx plat

kubens kdp-system


# generate kcp admin kubeconfig

# https://github.com/kcp-dev/helm-charts/tree/main/charts/kcp#initial-access


kubectl get secret kcp-front-proxy-cert -o=jsonpath='{.data.tls\.crt}' | base64 -d > /training/.secrets/kcp-front-proxy-cert-ca.crt

kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-admin.yaml config set-cluster base --server https://internal.$DOMAIN:8443 --certificate-authority=/training/.secrets/kcp-front-proxy-cert-ca.crt
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-admin.yaml config set-cluster root --server https://internal.$DOMAIN:8443/clusters/root --certificate-authority=/training/.secrets/kcp-front-proxy-cert-ca.crt

kubectl apply -f /training/platform-cluster/kcp-admin-cert.yaml

kubectl get certs -A

kubectl get secret cluster-admin-client-cert -o=jsonpath='{.data.tls\.crt}' | base64 -d > /training/.secrets/cluster-admin-client-cert.crt
kubectl get secret cluster-admin-client-cert -o=jsonpath='{.data.tls\.key}' | base64 -d > /training/.secrets/cluster-admin-client-cert.key
chmod 600 /training/.secrets/cluster-admin-client-cert.crt /training/.secrets/cluster-admin-client-cert.key

kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-admin.yaml config set-credentials kcp-admin --client-certificate=/training/.secrets/cluster-admin-client-cert.crt --client-key=/training/.secrets/cluster-admin-client-cert.key
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-admin.yaml config set-context base --cluster=base --user=kcp-admin
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-admin.yaml config set-context root --cluster=root --user=kcp-admin
kubectl --kubeconfig=/training/.secrets/kubeconfig-kcp-admin.yaml config use-context root

# merge the kubeconfigs

# TODO get certs into kubeconfig

yq e ".clusters |= map(select(.name == \"base\").name = \"kcp-base\")" -i /training/.secrets/kubeconfig-kcp-admin.yaml
yq e ".clusters |= map(select(.name == \"root\").name = \"kcp-root\")" -i /training/.secrets/kubeconfig-kcp-admin.yaml
yq e ".contexts |= map(select(.name == \"base\").context.cluster = \"kcp-base\")" -i /training/.secrets/kubeconfig-kcp-admin.yaml
yq e ".contexts |= map(select(.name == \"root\").context.cluster = \"kcp-root\")" -i /training/.secrets/kubeconfig-kcp-admin.yaml
yq e ".contexts |= map(select(.name == \"base\").name = \"kcp-base\")" -i /training/.secrets/kubeconfig-kcp-admin.yaml
yq e ".contexts |= map(select(.name == \"root\").name = \"kcp-root\")" -i /training/.secrets/kubeconfig-kcp-admin.yaml
KUBECONFIG="/training/.secrets/kubeconfig-platform-cluster.yaml:/training/.secrets/kubeconfig-service-cluster.yaml:/training/.secrets/kubeconfig-kcp-admin.yaml" kubectl config view --raw > /training/.secrets/kubeconfig.yaml

# verify
kubectx

```
