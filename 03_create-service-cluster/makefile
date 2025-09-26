SHELL = /bin/bash

.PHONY: prepare-tf-config
prepare-tf-config:
	cp /training/kubeone_${K1_VERSION}_linux_amd64/examples/terraform/gce/*.tf /training/service-cluster/tf_infra/
	sed -i "s/<FILL-IN-YOUR-GCE-PROJECT-ID>/${GCE_PROJECT}/g" /training/service-cluster/tf_infra/terraform.tfvars
	sed -i "s/<FILL-IN-CLUSTER-NAME>/${TRAINEE_NAME}-service-cluster/g" /training/service-cluster/tf_infra/terraform.tfvars

.PHONY: terraform
terraform: prepare-tf-config
	terraform -chdir=/training/service-cluster/tf_infra init
	terraform -chdir=/training/service-cluster/tf_infra apply -var=control_plane_target_pool_members_count=1 -auto-approve
	
.PHONY: create-cluster
create-cluster: terraform
	cd /training/service-cluster; kubeone apply -t /training/service-cluster/tf_infra -y
	cp /training/service-cluster/${TRAINEE_NAME}-service-cluster-kubeconfig /training/.secrets/kubeconfig-service-cluster.yaml
	cd /training/service-cluster; kubeone config machinedeployments -m /training/service-cluster/kubeone.yaml -t /training/service-cluster/tf_infra > /training/service-cluster/md.yaml
	sed -i 's/cluster-api-autoscaler-node-group-max-size: "1"/cluster-api-autoscaler-node-group-max-size: "3"/g' /training/service-cluster/md.yaml
	kubectl apply -f /training/service-cluster/md.yaml
	kubectl -n kube-system wait md/${TRAINEE_NAME}-service-cluster-pool1 --for=jsonpath='{.status.readyReplicas}'=1 --timeout=300s
	kubectl wait --for=condition=Ready node --all --timeout=300s
