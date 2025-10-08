# Setup Platform

## Login into Kubermatic Helm Chart Repo

<!-- TODO quay secrets -->

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
# TODO do it in helmfile, problems with oci repos
helm registry login quay.io -u $KUBERMATIC_REGISTRY_USERNAME -p $KUBERMATIC_REGISTRY_PASSWORD
```

<!-- TODO verify pods running on all helm charts -->

## Install Developer Platform

```bash
# set <DOMAIN>
sed -i "s/<DOMAIN>/$DOMAIN/g" /training/platform-cluster/helm/values_developer-platform.yaml

# set <PULL_CREDENTIALS>
# TODO https://quay.io/repository/kubermatic/developer-platform
PULL_CREDENTIALS=<FILL-IN-PULL-CREDENTIALS>
echo "export PULL_CREDENTIALS=$PULL_CREDENTIALS" >> /root/.trainingrc
source /root/.trainingrc
sed -i "s/<PULL_CREDENTIALS>/$PULL_CREDENTIALS/g" /training/platform-cluster/helm/values_developer-platform.yaml

# apply developer-platform helm chart
helmfile sync --file /training/platform-cluster/helm/helmfile.yaml --selector id=developer-platform
```

## Install Developer Platform Dashboard

```bash
# set <DOMAIN>
sed -i "s/<DOMAIN>/$DOMAIN/g" /training/platform-cluster/helm/values_developer-platform-dashboard.yaml

# set <PULL_CREDENTIALS>
sed -i "s/<PULL_CREDENTIALS>/$PULL_CREDENTIALS/g" /training/platform-cluster/helm/values_developer-platform-dashboard.yaml

# set <OIDC_CLIENT_SECRET>
yq ".dashboard.config.authentication.oidc.clientSecret = \"$OIDC_CLIENT_SECRET\"" -i /training/platform-cluster/helm/values_developer-platform-dashboard.yaml

# set <SESSION_ENCRYPTION_KEY>
SESSION_ENCRYPTION_KEY=$(cat /dev/urandom | base64 | tr -dc 'A-Za-z0-9' | head -c32)
echo "export SESSION_ENCRYPTION_KEY=$SESSION_ENCRYPTION_KEY" >> /root/.trainingrc
source /root/.trainingrc
yq ".dashboard.config.authentication.encryptionKey = \"$SESSION_ENCRYPTION_KEY\"" -i /training/platform-cluster/helm/values_developer-platform-dashboard.yaml

# apply developer-platform-dashboard helm chart
helmfile sync --file /training/platform-cluster/helm/helmfile.yaml --selector id=developer-platform-dashboard
```

## Verify Platform

```bash
# visit the following url in your browser
echo https://dashboard.$DOMAIN

# get the email address
# TODO
# echo $TRAINEE_EMAIL
echo admin

# get the password
echo $PASSWORD
```
