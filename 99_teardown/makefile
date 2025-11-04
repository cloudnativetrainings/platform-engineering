SHELL = /bin/bash

.PHONY teardown:
teardown:
	kubeone reset -m /training/provider-cluster/kubeone.yaml -t /training/provider-cluster/tf_infra/ -y
	terraform -chdir=/training/provider-cluster/tf_infra/ destroy -auto-approve
	kubeone reset -m /training/platform-cluster/kubeone.yaml -t /training/platform-cluster/tf_infra/ -y
	terraform -chdir=/training/platform-cluster/tf_infra/ destroy -auto-approve
	gcloud dns record-sets delete *.${PLATFORM_DOMAIN}. --type=A --zone=${DNS_ZONE_NAME}
	gcloud dns record-sets delete internal.${PLATFORM_DOMAIN}. --type=A --zone=${DNS_ZONE_NAME}
	gcloud dns record-sets delete ${PLATFORM_DOMAIN}. --type=A --zone=${DNS_ZONE_NAME}