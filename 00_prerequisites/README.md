# Prerequisites

In this lab you will ensure everything is in place for the later labs.

## Copy your Training Files

```bash
# create a directory for holding sensitive information
mkdir /training/.secrets

# drag and drop the provided files named "gcloud-service-account.json" and ".trainingrc" into the folder `/training/.secrets/`

# merge the .trainingrc files
cat /training/.secrets/.trainingrc >> /root/.trainingrc 

# ensure the kubeconfig is set properly
echo 'export KUBECONFIG=/training/.secrets/kubeconfig.yaml' >> /root/.trainingrc

# ensure changes are applied in your current bash
source /root/.trainingrc
```

## Prepare your environment

```bash
# create a ssh key pair
make -C /training/00_prerequisites/ ssh

# ensure gce configuration properly setup
make -C /training/00_prerequisites/ gce

# add google credentials
echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc
```

## Verify your environment

```bash
# ensure all environment variables get set in your current bash
source /root/.trainingrc

# verify
make verify
```
