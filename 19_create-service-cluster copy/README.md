
# Create Service Cluster

<!-- TODO
install kcp - not in image
play around with kcp on first lab
 -->

```bash
# create the cluster
make -C /training/03_create-service-cluster create-cluster

# rename the kubernetes user for avoiding naming conflicts
yq e ".users |= map(select(.name == \"kubernetes-admin\").name = \"kubernetes-admin@platform-cluster\")" -i /training/.secrets/kubeconfig-platform-cluster.yaml
yq e ".contexts |= map(select(.context.user == \"kubernetes-admin\").context.user = \"kubernetes-admin@platform-cluster\")" -i /training/.secrets/kubeconfig-platform-cluster.yaml
yq e ".users |= map(select(.name == \"kubernetes-admin\").name = \"kubernetes-admin@service-cluster\")" -i /training/.secrets/kubeconfig-service-cluster.yaml
yq e ".contexts |= map(select(.context.user == \"kubernetes-admin\").context.user = \"kubernetes-admin@service-cluster\")" -i /training/.secrets/kubeconfig-service-cluster.yaml


# merge the two kubeconfigs into one
KUBECONFIG=/training/.secrets/kubeconfig-platform-cluster.yaml:/training/.secrets/kubeconfig-service-cluster.yaml kubectl config view --raw > /training/.secrets/kubeconfig.yaml

# # rename the kubeconfig contexts
# TODO MAYBE BETTER WITHOUT RENAMING IT
# kubectl config rename-context kubernetes-admin@${TRAINEE_NAME}-platform-cluster platform-cluster
# kubectl config rename-context kubernetes-admin@${TRAINEE_NAME}-service-cluster service-cluster

# verify your kubeconfig
kubectx

# switch to service-cluster
kubectx service-cluster

# verify service cluster is working
kubectl get nodes

# TODO do i need this on service cluster
# cd /training/service-cluster; kubeone config machinedeployments -m /training/service-cluster/kubeone.yaml -t /training/service-cluster/tf_infra > /training/service-cluster/md.yaml
# sed -i 's/cluster-api-autoscaler-node-group-max-size: "1"/cluster-api-autoscaler-node-group-max-size: "3"/g' /training/service-cluster/md.yaml
# kubectl apply -f /training/service-cluster/md.yaml
# kubectl -n kube-system wait md/${TRAINEE_NAME}-service-cluster-pool1 --for=jsonpath='{.status.readyReplicas}'=1 --timeout=300s
# kubectl wait --for=condition=Ready node --all --timeout=300s
```

## Apply CRD

```bash
# create the crd
# ! on the service cluster
kubectl apply -f ./service-cluster/myservice/myservice_crd.yaml

# verify
# ! on the service cluster
kubectl get crd myservices.myorg.com
```
<!-- 
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

``` -->
