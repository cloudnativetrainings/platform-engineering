# Prerequisites

In this lab you will ensure everything is in place.

## Copy your Training Files

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

```bash
# create a ssh key pair
make -C /training/00_prerequisites/ ssh

# ensure gce configuration properly setup
make -C /training/00_prerequisites/ gce

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
