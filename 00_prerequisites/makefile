.PHONY: ssh
ssh:
	ssh-keygen -q -N "" -t rsa -f /training/.secrets/gce -C root
	chmod 400 /training/.secrets/gce
	echo 'eval `ssh-agent`' >> /root/.trainingrc
	echo "ssh-add /training/.secrets/gce" >> /root/.trainingrc

.PHONY: gce
gce: export GOOGLE_CREDENTIALS='$(shell cat /training/.secrets/gcloud-service-account.json)'
gce:
	gcloud auth activate-service-account --key-file=/training/.secrets/gcloud-service-account.json
	gcloud config set project ${GCE_PROJECT} --quiet
	gcloud config set compute/region europe-west3
	gcloud config set compute/zone europe-west3-a
	echo "export GOOGLE_CREDENTIALS=${GOOGLE_CREDENTIALS}" | tee -a /root/.trainingrc

.PHONY: install-k1
install-k1: export K1_VERSION=1.11.1
install-k1:
	echo "export K1_VERSION=${K1_VERSION}" | tee -a /root/.trainingrc
	wget -P /tmp/ https://github.com/kubermatic/kubeone/releases/download/v${K1_VERSION}/kubeone_${K1_VERSION}_linux_amd64.zip
	unzip /tmp/kubeone_${K1_VERSION}_linux_amd64.zip -d /training/kubeone_${K1_VERSION}_linux_amd64
	cp /training/kubeone_${K1_VERSION}_linux_amd64/kubeone /usr/local/bin
	echo 'source <(kubeone completion bash)' | tee -a /root/.trainingrc 
	
.PHONY: gcex
# gcex: export GOOGLE_CREDENTIALS=$(shell cat /training/.secrets/gcloud-service-account.json)
# gcex: ESCAPED_GOOGLE_CREDENTIALS=$(shell echo "$(GOOGLE_CREDENTIALS)" | shell sed 's/"/\\"/g')
gcex:
	echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc
# 	echo "export GOOGLE_CREDENTIALS=${GOOGLE_CREDENTIALS}" | tee -a /root/.trainingrc
# 	echo "export GOOGLE_CREDENTIALS=${GOOGLE_CREDENTIALS}" | tee -a /root/.trainingrc
# 	echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc
