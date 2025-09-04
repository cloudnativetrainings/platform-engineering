
.PHONY: prepare-tf-config
prepare-tf-config:
	cp /training/kubeone_${K1_VERSION}_linux_amd64/examples/terraform/gce/*.tf /training/platform-cluster/tf_infra/
	sed -i "s/<FILL-IN-YOUR-GCE-PROJECT-ID>/${GCE_PROJECT}/g" /training/platform-cluster/tf_infra/terraform.tfvars
	sed -i "s/<FILL-IN-CLUSTER-NAME>/${TRAINEE_NAME}-platform-cluster/g" /training/platform-cluster/tf_infra/terraform.tfvars

.PHONY: terraform
terraform: prepare-tf-config
	terraform -chdir=/training/platform-cluster/tf_infra init
	terraform -chdir=/training/platform-cluster/tf_infra apply -var=control_plane_target_pool_members_count=1 -auto-approve
	
.PHONY: create-cluster
create-cluster: terraform
	cd /training/platform-cluster; kubeone apply -t /training/platform-cluster/tf_infra -y --verbose
	terraform -chdir=/training/platform-cluster/tf_infra apply -auto-approve
	cd /training/platform-cluster; kubeone config machinedeployments -m /training/platform-cluster/kubeone.yaml -t /training/platform-cluster/tf_infra > /training/platform-cluster/md.yaml
	sed -i 's/cluster-api-autoscaler-node-group-max-size: "1"/cluster-api-autoscaler-node-group-max-size: "3"/g' /training/platform-cluster/md.yaml
	kubectl apply -f /training/platform-cluster/md.yaml
