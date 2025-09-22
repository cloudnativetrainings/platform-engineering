
# Create Platform Cluster

## Create Kubernetes Cluster

```bash
# create the cluster
make -C /training/01_create-platform-cluster create-cluster
```

## Install Infra Components on Kuberentes Cluster

```bash
# install storageclass
kubectl apply -f /training/platform-cluster/storageclass.yaml

# install ingess-nginx helm chart
helmfile sync --file /training/platform-cluster/helm/helm/helmfile.yaml --selector id=ingress-nginx

# install cert-manager helm chart
helmfile sync --file /training/platform-cluster/helm/helm/helmfile.yaml --selector id=cert-manager
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
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="$DOMAIN." --type A $INGRESS_IP
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="*.$DOMAIN." --type A $INGRESS_IP
gcloud dns record-sets transaction execute --zone $DNS_ZONE_NAME

# verify via nslookup
nslookup $DOMAIN
nslookup test.$DOMAIN

# apply clusterissuers
sed -i "s/your-email@example.com/$TRAINEE_EMAIL/g" /training/platform-cluster/cluster-issuer_letsencrypt-staging.yaml
sed -i "s/your-email@example.com/$TRAINEE_EMAIL/g" /training/platform-cluster/cluster-issuer_letsencrypt-prod.yaml
kubectl apply -f /training/platform-cluster/cluster-issuer_letsencrypt-prod.yaml
kubectl apply -f /training/platform-cluster/cluster-issuer_letsencrypt-staging.yaml
```
