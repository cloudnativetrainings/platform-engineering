SHELL = /bin/bash

.PHONY: ssh
ssh:
	ssh-keygen -q -N "" -t rsa -f /training/.secrets/gcp -C root
	chmod 400 /training/.secrets/gcp
	echo 'eval `ssh-agent`' >> /root/.trainingrc
	echo "ssh-add /training/.secrets/gcp" >> /root/.trainingrc

.PHONY: gcp
gcp:
	gcloud auth activate-service-account --key-file=/training/.secrets/gcloud-service-account.json
	gcloud config set project ${GCP_PROJECT} --quiet
	gcloud config set compute/region europe-west3
	gcloud config set compute/zone europe-west3-a
	