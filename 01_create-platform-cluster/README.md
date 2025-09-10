
```bash
# create the cluster
make -C /training/platform-cluster/ create-cluster


# TODO

# sudo curl -LO  https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz \
#   && sudo tar -zxvf helm-v${HELM_VERSION}-linux-amd64.tar.gz \
#   && sudo install -o root -g root -m 0755 linux-amd64/helm /usr/local/bin/helm \
#   && sudo rm helm-v${HELM_VERSION}-linux-amd64.tar.gz \
#   && sudo rm -rf linux-amd64/ \
#   && echo 'source <(helm completion bash)' | sudo tee -a /root/.trainingrc

# rm -rf /root/.local/share/helm/plugins/helm-diff/
# helm plugin install https://github.com/databus23/helm-diff


# install kdp
make -C /training/platform-cluster/ install-kdp
```

```bash


### Create proper DNS entry for the ingress controller

# persist the IP address of the nginx inress controller loadbalancer
echo "export INGRESS_IP=$(kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')" >> /root/.trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc

# verify
echo $INGRESS_IP

# create wildcard DNS entry
gcloud dns record-sets transaction start --zone $DNS_ZONE_NAME
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="$DOMAIN." --type A $INGRESS_IP
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="*.$DOMAIN." --type A $INGRESS_IP
gcloud dns record-sets transaction execute --zone $DNS_ZONE_NAME

# verify via nslookup
nslookup $DOMAIN
nslookup test.$DOMAIN

```
