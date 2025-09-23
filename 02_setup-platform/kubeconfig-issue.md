
## oidc

```bash
# download kcp admin kubeconfig named "./.secrets/kubeconfig-kcp-admin.yaml"

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

kubectl krew list

kubectl krew index add kcp-dev https://github.com/kcp-dev/krew-index.git
kubectl krew install kcp-dev/kcp
kubectl krew install kcp-dev/ws
kubectl krew install kcp-dev/create-workspace
 
# create callback url
echo https://$CODESPACE_NAME-8000.app.github.dev

# add callback url to dex helm
helmfile sync --file /training/platform-cluster/helm/helmfile.yaml --selector id=dex

# add --oidc-redirect-url=https://obscure-space-succotash-gx66rgjvg9q3944p-8000.app.github.dev to downloaded kubeconfig

kubectl --kubeconfig=./kubeconfig-kcp-admin.yaml get workspaces
```

## merge kubeconfigs

```bash
KUBECONFIG=/training/.secrets/kubeconfig-kcp-admin.yaml:/training/.secrets/kubeconfig-kcp-myorg-com-myservice.yaml:/training/.secrets/kubeconfig-platform-cluster.yaml:/training/.secrets/kubeconfig-service-cluster.yaml kubectl config view --flatten > /training/.secrets/kubeconfig.yaml

export KUBECONFIG=/training/.secrets/kubeconfig.yaml
```
