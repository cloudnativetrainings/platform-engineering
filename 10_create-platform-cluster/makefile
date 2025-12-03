SHELL = /bin/bash

.PHONY: prepare-tf-config
prepare-tf-config:
	cp /root/kubeone_${K1_VERSION}_linux_amd64/examples/terraform/gce/*.tf /training/platform-cluster/tf_infra/
	sed -i "s/<FILL-IN-YOUR-GCE-PROJECT-ID>/${GCE_PROJECT}/g" /training/platform-cluster/tf_infra/terraform.tfvars
	sed -i "s/<FILL-IN-CLUSTER-NAME>/${TRAINEE_NAME}-platform-cluster/g" /training/platform-cluster/tf_infra/terraform.tfvars

.PHONY: terraform
terraform: prepare-tf-config
	terraform -chdir=/training/platform-cluster/tf_infra init
	terraform -chdir=/training/platform-cluster/tf_infra apply -var=control_plane_target_pool_members_count=1 -auto-approve
	
.PHONY: create-cluster
create-cluster: terraform
	cd /training/platform-cluster; kubeone apply -t /training/platform-cluster/tf_infra -y
	cp /training/platform-cluster/${TRAINEE_NAME}-platform-cluster-kubeconfig /training/.secrets/kubeconfig-platform-cluster.yaml
	yq e ".users |= map(select(.name == \"kubernetes-admin\").name = \"admin@k8s-platform\")" -i /training/.secrets/kubeconfig-platform-cluster.yaml
	yq e ".contexts |= map(select(.context.user == \"kubernetes-admin\").context.user = \"admin@k8s-platform\")" -i /training/.secrets/kubeconfig-platform-cluster.yaml
	yq ".contexts[0].name = \"admin@k8s-platform\"" -i /training/.secrets/kubeconfig-platform-cluster.yaml	
	yq ".current-context = \"admin@k8s-platform\"" -i /training/.secrets/kubeconfig-platform-cluster.yaml
	make -C /training squash-kubeconfigs
	cd /training/platform-cluster; kubeone config machinedeployments -m /training/platform-cluster/kubeone.yaml -t /training/platform-cluster/tf_infra > /training/platform-cluster/md.yaml
	sed -i 's/cluster-api-autoscaler-node-group-max-size: "1"/cluster-api-autoscaler-node-group-max-size: "3"/g' /training/platform-cluster/md.yaml
	kubectl apply -f /training/platform-cluster/md.yaml
	kubectl -n kube-system wait md/${TRAINEE_NAME}-platform-cluster-pool1 --for=jsonpath='{.status.readyReplicas}'=1 --timeout=300s
	kubectl wait --for=condition=Ready node --all --timeout=300s
