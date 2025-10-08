# Setup Platform

## Install DEX

```bash

# TODO why in namespace kdp-system

# TDOD has to be done in platform cluster

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
# TODO place this somewhere else in lab 02 setup platform
# get the redirect url
echo https://$CODESPACE_NAME-8000.app.github.dev
yq e ".config.staticClients[0].RedirectURIs += \"https://$CODESPACE_NAME-8000.app.github.dev\"" -i /training/platform-cluster/helm/values_dex.yaml

# apply dex helm chart
helmfile sync --file /training/platform-cluster/helm/helmfile.yaml --selector id=dex

# wait until dex certificate gets valid
watch -n 1 kubectl -n kdp-system get certs

# visit dex in the browser
echo https://login.$DOMAIN
```

## Install KCP

```bash

# TODO why in namespace kdp-system




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

# TODO why did i need dex upfront? there is no ingress for front proxy

# verify via nslookup
nslookup internal.$DOMAIN
```
