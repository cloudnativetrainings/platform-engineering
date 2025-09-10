# Prerequisites

In this lab you will ensure everything is in place.

## Copy your Training Files

<!-- TODO

.trainingrc alredy looks like this
PS1="\[[0;32m\]\u@\H \[[0;34m\]\w >[0m "
cd /training/
source <(kubectl completion bash)
export K8S_VERSION=1.32.4
export TF_VERSION=1.12.2-1
source /root/kubectx.bash
source /root/kubens.bash
source <(helm completion bash)
 -->

```bash
# create a directory for holding sensitive information
mkdir /training/.secrets

# drag and drop the provided file named "gcloud-service-account.json" into the folder `/training/.secrets/`

# drag and drop the provided file named ".trainingrc" into the folder `/training/`

# move the file named ".trainingrc" into the root folder
mv /training/.trainingrc /root/

# ensure changes are applied in your current bash
source /root/.trainingrc
```

## Prepare your environment

<!-- TODO google credentials file does not work

export GOOGLE_CREDENTIALS=''

 -->

```bash
# create a ssh key pair
make -C /training/00_prerequisites/ ssh

# ensure gce configuration properly setup
make -C /training/00_prerequisites/ gce

# TODO
echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc

# install k1
make -C /training/00_prerequisites/ install-k1
```

## Verify your environment

```bash
# ensure all environment variables get set in your current bash
source /root/.trainingrc

# verify
make verify
```
