# Prerequisites

In this lab you will ensure everything is in place.

## Copy your Training Files

<!-- TODO in /root/.trainingrc env substitution
export K1_VERSION=${K1_VERSION}
source <(kubeone completion bash)
export PATH="/root/.krew/bin:${PATH}"
export KREW_VERSION=${KREW_VERSION}
 -->

```bash
# create a directory for holding sensitive information
mkdir /training/.secrets

# drag and drop the provided file named "gcloud-service-account.json" into the folder `/training/.secrets/`

# merge the .trainingrc files
cat .trainingrc >> /root/.trainingrc 
rm .trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc
```

## Prepare your environment

```bash
# create a ssh key pair
make -C /training/00_prerequisites/ ssh

# ensure gce configuration properly setup
make -C /training/00_prerequisites/ gce

# TODO
echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc

# TODO training-infra k1 dir in /training does not exist
#  echo "export K1_VERSION=${K1_VERSION}" | tee -a /root/.trainingrc
#  wget -P /tmp/ https://github.com/kubermatic/kubeone/releases/download/v${K1_VERSION}/kubeone_${K1_VERSION}_linux_amd64.zip
#  unzip /tmp/kubeone_${K1_VERSION}_linux_amd64.zip -d /training/kubeone_${K1_VERSION}_linux_amd64
#  cp /training/kubeone_${K1_VERSION}_linux_amd64/kubeone /usr/local/bin
#  echo 'source <(kubeone completion bash)' | tee -a /root/.trainingrc 
```

<!-- TODO fix domain name platform cluster and service cluster -->

## Verify your environment

```bash
# ensure all environment variables get set in your current bash
source /root/.trainingrc

# verify
make verify
```
