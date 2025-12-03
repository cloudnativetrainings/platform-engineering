SHELL = /bin/bash

.PHONY: prepare-tf-config
prepare-tf-config:
	cp /root/kubeone_${K1_VERSION}_linux_amd64/examples/terraform/gce/*.tf /training/provider-cluster/tf_infra/
	sed -i "s/<FILL-IN-YOUR-GCE-PROJECT-ID>/${GCE_PROJECT}/g" /training/provider-cluster/tf_infra/terraform.tfvars
	sed -i "s/<FILL-IN-CLUSTER-NAME>/${TRAINEE_NAME}-provider-cluster/g" /training/provider-cluster/tf_infra/terraform.tfvars

.PHONY: terraform
terraform: prepare-tf-config
	terraform -chdir=/training/provider-cluster/tf_infra init
	terraform -chdir=/training/provider-cluster/tf_infra apply -var=control_plane_target_pool_members_count=1 -auto-approve
	
.PHONY: create-cluster
create-cluster: terraform
	cd /training/provider-cluster; kubeone apply -t /training/provider-cluster/tf_infra -y
	cp /training/provider-cluster/${TRAINEE_NAME}-provider-cluster-kubeconfig /training/.secrets/kubeconfig-provider-cluster.yaml
	yq e ".users |= map(select(.name == \"kubernetes-admin\").name = \"admin@k8s-provider\")" -i /training/.secrets/kubeconfig-provider-cluster.yaml
	yq e ".contexts |= map(select(.context.user == \"kubernetes-admin\").context.user = \"admin@k8s-provider\")" -i /training/.secrets/kubeconfig-provider-cluster.yaml
	yq ".contexts[0].name = \"admin@k8s-provider\"" -i /training/.secrets/kubeconfig-provider-cluster.yaml	
	yq ".current-context = \"admin@k8s-provider\"" -i /training/.secrets/kubeconfig-provider-cluster.yaml
	make -C /training squash-kubeconfigs
	kubectx admin@k8s-provider
	cd /training/provider-cluster; kubeone config machinedeployments -m /training/provider-cluster/kubeone.yaml -t /training/provider-cluster/tf_infra > /training/provider-cluster/md.yaml
	sed -i 's/cluster-api-autoscaler-node-group-max-size: "1"/cluster-api-autoscaler-node-group-max-size: "3"/g' /training/provider-cluster/md.yaml
	kubectl apply -f /training/provider-cluster/md.yaml
	kubectl -n kube-system wait md/${TRAINEE_NAME}-provider-cluster-pool1 --for=jsonpath='{.status.readyReplicas}'=1 --timeout=300s
	kubectl wait --for=condition=Ready node --all --timeout=300s
