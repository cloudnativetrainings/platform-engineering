# Teardown the infrastructure

In this lab you will destroy all infrastructure you have created.

```bash
# reset the cluster
kubeone reset -m /training/platform-cluster/kubeone.yaml -t /training/platform-cluster/tf_infra/ -y

# destroy the infrastructure provided via terraform
terraform -chdir=/training/platform-cluster/tf_infra/ destroy -auto-approve

# delete the gce DNS entries
gcloud dns record-sets delete *.$DOMAIN. --type=A --zone=$DNS_ZONE_NAME
gcloud dns record-sets delete $DOMAIN. --type=A --zone=$DNS_ZONE_NAME

```
