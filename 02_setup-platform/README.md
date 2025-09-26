# Setup Platform

## Install DEX

```bash
# set <DOMAIN>
sed -i "s/<DOMAIN>/$DOMAIN/g" /training/platform-cluster/helm/values_dex.yaml

# set <OIDC_CLIENT_SECRET>
OIDC_CLIENT_SECRET=$(cat /dev/urandom | base64 | tr -dc 'A-Za-z0-9' | head -c32)
echo "export OIDC_CLIENT_SECRET=$OIDC_CLIENT_SECRET" >> /root/.trainingrc
source /root/.trainingrc
yq ".config.staticClients[0].secret = \"$OIDC_CLIENT_SECRET\"" -i /training/platform-cluster/helm/values_dex.yaml

# set <ADMIN_PASSWORD_HASH>
PASSWORD=<FILL-IN-YOUR-PASSWORD>
PASSWORD_HASH=$(echo "$PASSWORD" | htpasswd -inBC 10 admin | cut -d: -f2)
echo "export PASSWORD=$PASSWORD" >> /root/.trainingrc
echo "export PASSWORD_HASH='$PASSWORD_HASH'" >> /root/.trainingrc
source /root/.trainingrc
yq ".config.staticPasswords[0].hash = \"$PASSWORD_HASH\"" -i /training/platform-cluster/helm/values_dex.yaml

# TODO add the redirect uri madness here!!!

# apply dex helm chart
helmfile sync --file /training/platform-cluster/helm/helmfile.yaml --selector id=dex

# wait until dex certificate gets valid
watch -n 1 kubectl -n kdp-system get certs

# visit dex in the browser
echo https://login.$DOMAIN
```

## Install KCP

```bash
# set <DOMAIN>
sed -i "s/<DOMAIN>/$DOMAIN/g" /training/platform-cluster/helm/values_kcp.yaml

# apply kcp helm chart
helmfile sync --file /training/platform-cluster/helm/helmfile.yaml --selector id=kcp

# persist the IP address of the kcp-front-proxy loadbalancer
echo "export KCP_FRONT_PROXY_IP=$(kubectl -n kdp-system get svc kcp-front-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')" >> /root/.trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc

# verify
echo $KCP_FRONT_PROXY_IP

# create kcp DNS entry
gcloud dns record-sets transaction start --zone $DNS_ZONE_NAME
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="internal.$DOMAIN." --type A $KCP_FRONT_PROXY_IP
gcloud dns record-sets transaction execute --zone $DNS_ZONE_NAME

# verify via nslookup
nslookup internal.$DOMAIN
```

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
