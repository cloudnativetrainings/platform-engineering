# Setup KDP

In this lab you will setup KDP.

```bash
# switch to the platform cluster
kubectx admin@k8s-platform
```

## Preparations

### DEX

KDP supports you in the kubeconfig madness. Therefore we have to adapt a few things.

```bash
# install the oidc-login krew plugin
kubectl krew install oidc-login

# add an redirect URI to dex; this is needed only due to the training platform
yq e ".config.staticClients[0].RedirectURIs += \"https://$CODESPACE_NAME-8000.app.github.dev\"" -i /training/12_setup-kcp-in-platform-cluster/helm/values_dex.yaml

# release the changes in dex
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=dex
```

### KCP

Due to a breaking change in kcp we have to engage a feature gate in it. [The issue](https://github.com/kubermatic/developer-platform/issues/334) will be fixed.

Add the following to the kcp helm values file `/training/12_setup-kcp-in-platform-cluster/helm/values_kcp.yaml`

```yaml
kcp:
  extraFlags:
    - '--feature-gates=EnableDeprecatedAPIExportVirtualWorkspacesUrls=true'
```

Re-release kcp

```bash
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=kcp
```

## Login into Kubermatic Helm Chart Repo

<!-- TODO quay secrets -->
<!-- https://quay.io/repository/kubermatic/developer-platform -->

```bash
# set <KUBERMATIC_REGISTRY_USERNAME>
KUBERMATIC_REGISTRY_USERNAME=<FILL-IN-USERNAME>
echo "export KUBERMATIC_REGISTRY_USERNAME=$KUBERMATIC_REGISTRY_USERNAME" >> /root/.trainingrc
source /root/.trainingrc

# set <KUBERMATIC_REGISTRY_PASSWORD>
KUBERMATIC_REGISTRY_PASSWORD=<FILL-IN-PASSWORD>
echo "export KUBERMATIC_REGISTRY_PASSWORD=$KUBERMATIC_REGISTRY_PASSWORD" >> /root/.trainingrc
source /root/.trainingrc

# login into kubermatic repo
helm registry login quay.io -u $KUBERMATIC_REGISTRY_USERNAME -p $KUBERMATIC_REGISTRY_PASSWORD
```

## Install Developer Platform

```bash
# set <DOMAIN>
sed -i "s/<DOMAIN>/$PLATFORM_DOMAIN/g" /training/40_setup-kdp/helm/values_developer-platform.yaml

# set <PULL_CREDENTIALS>
# TODO
PULL_CREDENTIALS=<FILL-IN-PULL-CREDENTIALS>
echo "export PULL_CREDENTIALS=$PULL_CREDENTIALS" >> /root/.trainingrc
source /root/.trainingrc
sed -i "s/<PULL_CREDENTIALS>/$PULL_CREDENTIALS/g" /training/40_setup-kdp/helm/values_developer-platform.yaml

# apply developer-platform helm chart
helmfile sync --file /training/40_setup-kdp/helm/helmfile.yaml --selector id=developer-platform

# verify all pods are running
kubectl -n kcp-system get pods
```

## Install Developer Platform Dashboard

```bash
# set <DOMAIN>
sed -i "s/<DOMAIN>/$PLATFORM_DOMAIN/g" /training/40_setup-kdp/helm/values_developer-platform-dashboard.yaml

# set <PULL_CREDENTIALS>
sed -i "s/<PULL_CREDENTIALS>/$PULL_CREDENTIALS/g" /training/40_setup-kdp/helm/values_developer-platform-dashboard.yaml

# set <OIDC_CLIENT_SECRET>
yq ".dashboard.config.authentication.oidc.clientSecret = \"$OIDC_CLIENT_SECRET\"" -i /training/40_setup-kdp/helm/values_developer-platform-dashboard.yaml

# set <SESSION_ENCRYPTION_KEY>
SESSION_ENCRYPTION_KEY=$(cat /dev/urandom | base64 | tr -dc 'A-Za-z0-9' | head -c32)
echo "export SESSION_ENCRYPTION_KEY=$SESSION_ENCRYPTION_KEY" >> /root/.trainingrc
source /root/.trainingrc
yq ".dashboard.config.authentication.encryptionKey = \"$SESSION_ENCRYPTION_KEY\"" -i /training/40_setup-kdp/helm/values_developer-platform-dashboard.yaml

# apply developer-platform-dashboard helm chart
helmfile sync --file /training/40_setup-kdp/helm/helmfile.yaml --selector id=developer-platform-dashboard

# verify all pods are running
kubectl -n kcp-system get pods
```

## Verify Platform

```bash
# visit the following url in your browser
echo https://dashboard.$PLATFORM_DOMAIN

# login via user `admin` and the password you created earlier on setting up kcp
echo $PASSWORD
```
