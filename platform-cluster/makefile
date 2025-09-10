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
	cd /training/platform-cluster; kubeone apply -t /training/platform-cluster/tf_infra -y
	mkdir /root/.kube/; cp /training/platform-cluster/hubert-platform-cluster-kubeconfig /root/.kube/config
	terraform -chdir=/training/platform-cluster/tf_infra apply -auto-approve
	cd /training/platform-cluster; kubeone config machinedeployments -m /training/platform-cluster/kubeone.yaml -t /training/platform-cluster/tf_infra > /training/platform-cluster/md.yaml
	sed -i 's/cluster-api-autoscaler-node-group-max-size: "1"/cluster-api-autoscaler-node-group-max-size: "3"/g' /training/platform-cluster/md.yaml
	kubectl apply -f /training/platform-cluster/md.yaml

.PHONY: install-kdp
install-kdp: 
# 	TODO what if node does not exist yet
	kubectl wait --for=condition=Ready node --all
	kubectl apply -f /training/platform-cluster/storageclass.yaml
	helmfile sync

.PHONY: post
# TODO password from env var?
post: export ADMIN_PASSWORD='5aFePa5Sw0rD;'
post: export ADMIN_PASSWORD_HASH=$(shell echo 'YOUR_PASSWORD' | htpasswd -inBC 10 admin | cut -d: -f2)
post: export OIDC_CLIENT_SECRET=$(shell cat /dev/urandom | base64 | tr -dc 'A-Za-z0-9' | head -c32)
post:
	helm upgrade --install --atomic \
		ingress-nginx ingress-nginx \
		--version 4.12.2 \
  		--repo https://kubernetes.github.io/ingress-nginx \
  		--namespace ingress-nginx \
		--create-namespace \
		--values /training/platform-cluster/values_ingress-nginx.yaml
	sed -i "s/your-email@example.com/$(TRAINEE_EMAIL)/g" /training/platform-cluster/cluster-issuer_letsencrypt-prod.yaml
	kubectl apply -f /training/platform-cluster/cluster-issuer_letsencrypt-prod.yaml
	sed -i "s/<DOMAIN>/$(DOMAIN)/g" /training/platform-cluster/values_dex.yaml
	sed -i "s/<OIDC_CLIENT_SECRET>/"$(OIDC_CLIENT_SECRET)"/g" /training/platform-cluster/values_dex.yaml
	echo TODO BEGIN
	echo $(ADMIN_PASSWORD)
	echo "$(ADMIN_PASSWORD_HASH)"
# 	yq ".config.staticPasswords[0].hash = \"$(ADMIN_PASSWORD_HASH)\"" -i /training/platform-cluster/values_dex.yaml
# 	sed -i "s/<ADMIN_PASSWORD_HASH>/'$(ADMIN_PASSWORD_HASH)'/g" /training/platform-cluster/values_dex.yaml
	echo TODO END
	helm upgrade --install --atomic \
		dex dex \
		--repo=https://charts.dexidp.io \
		--version=0.23.0 \
		--create-namespace \
		--namespace=kdp-system \
		--values /training/platform-cluster/values_dex.yaml
	sed -i "s/<DOMAIN>/$(DOMAIN)/g" /training/platform-cluster/values_kcp.yaml
	helm upgrade --install --atomic \
		kcp kcp \
		--repo=https://kcp-dev.github.io/helm-charts \
		--version=0.11.1 \
		--create-namespace \
		--namespace=kdp-system \
		--values=/training/platform-cluster/values_kcp.yaml
