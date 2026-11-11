# Kubermatic Developer Platform

In this training you will learn how to install and administer the Kubermatic Developer Platform (KDP).

## Set up the training environment

### Clone the Git Repo

```bash
git clone https://github.com/cloudnativetrainings/platform-engineering
```

### Run the kdp-workshop Container

```bash
docker run -it -d \
  --name kdp-workshop \
  --restart=always \
  --cpus=2 \
  --memory=4g \
  -p 8000:8000 \
  -p 8080:8080 \
  --hostname kdp-workshop \
  -v $(PWD)/platform-engineering:/training \
  quay.io/kubermatic-labs/training-ghcs-platform-engineering-trainee-environment:2.0.0
```

### Access your kdp-workshop IDE with your browser

The URL therefore is <http://localhost:8080>

### Verify acceess to GCP from within the container

```bash
gcloud info --run-diagnostics 
```
