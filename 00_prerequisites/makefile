SHELL = /bin/bash

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
# 	echo "export GOOGLE_CREDENTIALS=${GOOGLE_CREDENTIALS}" | tee -a /root/.trainingrc
	
.PHONY: gcex
# gcex: export GOOGLE_CREDENTIALS=$(shell cat /training/.secrets/gcloud-service-account.json)
# gcex: ESCAPED_GOOGLE_CREDENTIALS=$(shell echo "$(GOOGLE_CREDENTIALS)" | shell sed 's/"/\\"/g')
gcex:
	echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc
# 	echo "export GOOGLE_CREDENTIALS=${GOOGLE_CREDENTIALS}" | tee -a /root/.trainingrc
# 	echo "export GOOGLE_CREDENTIALS=${GOOGLE_CREDENTIALS}" | tee -a /root/.trainingrc
# 	echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc
