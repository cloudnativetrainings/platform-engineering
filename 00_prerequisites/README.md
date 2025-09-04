# Prerequisites

In this lab you will ensure everything is in place to create a kubernetes cluster via kubeone.

## Verify installed software

```bash
# verify kubectl is installed
kubectl version --client

# verify terraform is installed
terraform version
```

### Copy your Training Files

```bash
# create a directory for holding sensitive information
mkdir /training/.secrets
```

Drag and Drop the files (provided by the trainer) into the directory `/training/.secrets/`

- README.md
- gcloud-service-account.json

## Set important environment variables

> **IMORTANT:**
> Those variables will get referenced during the following labs. Make sure to set them before continuing.
> You can find the needed information in the file `/training/.secrets/README.md`

```bash
# persist the project id into an environment variable
echo "export GCE_PROJECT=<FILL-IN-GOOGLE-PROJECT-ID>" >> /root/.trainingrc

# persist your trainee name into an environment variable
echo "export TRAINEE_NAME=<FILL-IN-TRAINEE-NAME>" >> /root/.trainingrc

# persist your domain into an environment variable
echo "export DOMAIN=<FILL-IN-DOMAIN>" >> /root/.trainingrc

# persist your domain into an environment variable
echo "export DNS_ZONE_NAME=<FILL-IN-DNS-ZONE-NAME>" >> /root/.trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc

# verify
echo $GCE_PROJECT
echo $TRAINEE_NAME
echo $DOMAIN
echo $DNS_ZONE_NAME
```

## Ensure SSH requirements

Kubeone needs a ssh key pair for communicating with the controlplane and worker nodes.

```bash
# create a ssh-key-pair for gce
ssh-keygen -q -N "" -t rsa -f /training/.secrets/gce -C root

# ensure proper private key file permissions
chmod 400 /training/.secrets/gce

# ensure .ssh key is known on environment restarts
echo 'eval `ssh-agent`' >> /root/.trainingrc
echo "ssh-add /training/.secrets/gce" >> /root/.trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc

# verify agent is running and holds proper key
ssh-add -l | grep "$(ssh-keygen -lf /training/.secrets/gce)"
```

## Configure GCE

```bash
# activate gce account
gcloud auth activate-service-account --key-file=/training/.secrets/gcloud-service-account.json

# set the gce project
gcloud config set project $GCE_PROJECT --quiet

# set the compute region and zone
gcloud config set compute/region europe-west3
gcloud config set compute/zone europe-west3-a

# verify your settings
gcloud config list

# persist the google credentials into an environment variable (needed by terraform and k1)
echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc
```

## Install KubeOne

```bash
# set the k1 version
K1_VERSION=1.11.1

# download the k1 release
wget -P /tmp/ https://github.com/kubermatic/kubeone/releases/download/v${K1_VERSION}/kubeone_${K1_VERSION}_linux_amd64.zip

# unzip k1 release
unzip /tmp/kubeone_${K1_VERSION}_linux_amd64.zip -d /training/kubeone_${K1_VERSION}_linux_amd64

# copy k1 into directory within `$PATH`
cp /training/kubeone_${K1_VERSION}_linux_amd64/kubeone /usr/local/bin

# verify k1 installation
kubeone version

# add k1 completion to your environment
echo 'source <(kubeone completion bash)' | tee -a /root/.trainingrc 

# persist the k1 version into an environment variable
echo "export K1_VERSION=${K1_VERSION}" | tee -a /root/.trainingrc
```

## Verify your environment

```bash
# ensure all environment variables get set in your current bash
source /root/.trainingrc

# verify
make verify
```
