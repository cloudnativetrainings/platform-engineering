
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

# TODO merge the kubeconfigs


export KUBECONFIG=/training/.secrets/kubeconfig-kcp-admin.yaml
kubectl  ws .

kubectl ws tree

kubectl create-workspace test

kubectl --kubeconfig=/training/.secrets/kubeconfig.yaml get pods
```
