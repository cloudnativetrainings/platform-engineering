# Setup Platform

In this lab you will install kcp and its dependencies into the platform cluster.

```bash
# switch to the platform cluster
kubectx admin@k8s-platform
```

## Install Infra Components on Kuberentes Cluster

```bash
# install storageclass
kubectl apply -f /training/platform-cluster/storageclass.yaml

# release the ingess-nginx helm chart
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=ingress-nginx

# release the cert-manager helm chart
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=cert-manager
```

## Finish LetsEncrypt setup

```bash
# persist the IP address of the nginx inress controller loadbalancer
echo "export INGRESS_IP=$(kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')" >> /root/.trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc

# verify
echo $INGRESS_IP

# create DNS entries
gcloud dns record-sets transaction start --zone $DNS_ZONE_NAME
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="$PLATFORM_DOMAIN." --type A $INGRESS_IP
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="*.$PLATFORM_DOMAIN." --type A $INGRESS_IP
gcloud dns record-sets transaction execute --zone $DNS_ZONE_NAME

# verify via nslookup
nslookup $PLATFORM_DOMAIN
nslookup test.$PLATFORM_DOMAIN

# apply clusterissuers
sed -i "s/your-email@example.com/$TRAINEE_EMAIL/g" /training/platform-cluster/cluster-issuer_letsencrypt-staging.yaml
sed -i "s/your-email@example.com/$TRAINEE_EMAIL/g" /training/platform-cluster/cluster-issuer_letsencrypt-prod.yaml
kubectl apply -f /training/platform-cluster/cluster-issuer_letsencrypt-prod.yaml
kubectl apply -f /training/platform-cluster/cluster-issuer_letsencrypt-staging.yaml
```

## Install DEX

```bash
# set <DOMAIN>
sed -i "s/<DOMAIN>/$PLATFORM_DOMAIN/g" /training/12_setup-kcp-in-platform-cluster/helm/values_dex.yaml

# set <OIDC_CLIENT_SECRET>
OIDC_CLIENT_SECRET=$(cat /dev/urandom | base64 | tr -dc 'A-Za-z0-9' | head -c32)
echo "export OIDC_CLIENT_SECRET=$OIDC_CLIENT_SECRET" >> /root/.trainingrc
source /root/.trainingrc
yq ".config.staticClients[0].secret = \"$OIDC_CLIENT_SECRET\"" -i /training/12_setup-kcp-in-platform-cluster/helm/values_dex.yaml

# set <ADMIN_PASSWORD_HASH>
PASSWORD=<FILL-IN-YOUR-PASSWORD>
PASSWORD_HASH=$(echo "$PASSWORD" | htpasswd -inBC 10 admin | cut -d: -f2)
echo "export PASSWORD=$PASSWORD" >> /root/.trainingrc
echo "export PASSWORD_HASH='$PASSWORD_HASH'" >> /root/.trainingrc
source /root/.trainingrc
yq ".config.staticPasswords[0].hash = \"$PASSWORD_HASH\"" -i /training/12_setup-kcp-in-platform-cluster/helm/values_dex.yaml

# release the dex helm chart
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=dex

# wait until dex certificate gets valid
watch -n 1 kubectl -n dex get certs

# visit dex in the browser, verify TLS is working
echo https://login.$PLATFORM_DOMAIN
```

## Install KCP

```bash
# set <DOMAIN>
sed -i "s/<DOMAIN>/$PLATFORM_DOMAIN/g" /training/12_setup-kcp-in-platform-cluster/helm/values_kcp.yaml

# release the kcp helm chart
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=kcp

# persist the IP address of the kcp-front-proxy loadbalancer
echo "export KCP_FRONT_PROXY_IP=$(kubectl -n kcp-system get svc kcp-front-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')" >> /root/.trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc

# verify
echo $KCP_FRONT_PROXY_IP

# create kcp DNS entry
gcloud dns record-sets transaction start --zone $DNS_ZONE_NAME
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="internal.$PLATFORM_DOMAIN." --type A $KCP_FRONT_PROXY_IP
gcloud dns record-sets transaction execute --zone $DNS_ZONE_NAME

# verify via nslookup
nslookup internal.$PLATFORM_DOMAIN

# verify pods are running
kubectl -n kcp-system get pods
```
