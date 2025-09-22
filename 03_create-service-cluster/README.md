
# Create Service Cluster

```bash
# start cluster
docker-desktop
kind delete cluster --name service-cluster
kind create cluster --kubeconfig .secrets/kubeconfig-service-cluster.yaml --config ./service-cluster/kind-service-cluster.yaml
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml get nodes

# create the crd
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml delete -f ./service-cluster/myservice_crd.yaml
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml apply -f ./service-cluster/myservice_crd.yaml
kubectl --kubeconfig=./.secrets/kubeconfig-service-cluster.yaml  get crds
```

## krew

```bash
# TODO do krew installation in container image
# TODO after oidc login is doable at github codespaces
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

kubectl krew install oidc-login

kubectl krew list
PLUGIN                    VERSION
kcp-dev/create-workspace  v0.28.1
kcp-dev/kcp               v0.28.1
kcp-dev/ws                v0.28.1
krew                      v0.4.5
oidc-login                v1.34.1


```
