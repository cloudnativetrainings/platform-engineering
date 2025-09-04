
```bash
# create the cluster
make -C /training/platform-cluster/ create-cluster

# ensure the downloaded kubeconfig is the default kubeconfig
mkdir /root/.kube
cp /training/platform-cluster/$TRAINEE_NAME-platform-cluster-kubeconfig /root/.kube/config



### Create proper DNS entry for the ingress controller

# persist the IP address of the nginx inress controller loadbalancer
echo "export INGRESS_IP=$(kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')" >> /root/.trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc

# verify
echo $INGRESS_IP

# create wildcard DNS entry
gcloud dns record-sets transaction start --zone $DNS_ZONE_NAME
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="platform.$DOMAIN." --type A $INGRESS_IP
gcloud dns record-sets transaction add --zone $DNS_ZONE_NAME --ttl 60 --name="*.platform.$DOMAIN." --type A $INGRESS_IP
gcloud dns record-sets transaction execute --zone $DNS_ZONE_NAME

# verify via nslookup
nslookup $DOMAIN
nslookup test.$DOMAIN

# change the email address in the manifest `cluster-issuer.yaml` to your email address
sed -i "s/your-email@example.com/$TRAINEE_EMAIL/g" /training/platform-cluster/cluster-issuer.yaml

# apply the cluster-issuer to your cluster
kubectl apply -f /training/platform-cluster/cluster-issuer.yaml

# verify certs
# kubectl get certs
```
