# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A hands-on Platform Engineering training built around **kcp** and **KDP** (Kubermatic Developer Platform). It is **not** application source code — it is a sequence of numbered labs (READMEs + supporting YAML/makefiles) that a trainee runs inside a prepared environment. Almost every command in the labs assumes the workspace is bind-mounted at the absolute path `/training/`, not the host's repo path.

## Runtime environment

The labs are designed for a GitHub Codespace using `.devcontainer/devcontainer.json`, which mounts the repo at `/training/` inside the image `quay.io/kubermatic-labs/training-ghcs-platform-engineering-trainee-environment:1.0.0` and runs as `root`. That image preinstalls `kubectl`, `helm`, `helmfile`, `terraform`, `kubeone`, `etcdctl`, `gcloud`, `yq`, `kubectx`/`kubens`, plus a kcp `kubectl krew` plugin set.

When editing or testing lab content, do not rewrite `/training/...` paths to relative paths — trainees copy/paste these commands verbatim, and the absolute path is part of the contract.

## Configuration model: `.trainingrc`

Lab steps repeatedly `source /root/.trainingrc` and append exports to it. This file is the single source of truth for per-trainee configuration. Required exports (verified by the root `makefile`'s `verify` target):

`GCE_PROJECT`, `TRAINEE_NAME`, `TRAINEE_EMAIL`, `DOMAIN`, `DNS_ZONE_NAME`, `K8S_VERSION`, `TF_VERSION`, `K1_VERSION`, `KUBECONFIG`, `PLATFORM_DOMAIN`, `PROVIDER_DOMAIN`, plus runtime-derived values such as `INGRESS_IP`, `KCP_FRONT_PROXY_IP`, `OIDC_CLIENT_SECRET`, `PASSWORD_HASH`, `GOOGLE_CREDENTIALS`.

Trainee-specific secrets live in `/training/.secrets/` (gitignored): the GCE service-account JSON, an ssh keypair, the trainee's `.trainingrc` fragment, and every generated `kubeconfig-*.yaml`.

## Common commands

```bash
# verify the environment is fully provisioned (env vars + tools + secrets present)
make verify

# merge all /training/.secrets/kubeconfig-*.yaml files into kubeconfig.yaml
make squash-kubeconfigs

# any lab with its own makefile is invoked with -C
make -C /training/<lab-dir> <target>
# e.g.
make -C /training/00_prerequisites    ssh
make -C /training/00_prerequisites    gce
make -C /training/10_create-platform-cluster create-cluster
make -C /training/11_create-provider-cluster create-cluster
make -C /training/99_teardown teardown   # full destroy (kubeone reset, tf destroy, DNS cleanup)

# helmfile is the install pattern for everything cluster-side; selectors pick one release
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=ingress-nginx
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=cert-manager
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=dex
helmfile sync --file /training/12_setup-kcp-in-platform-cluster/helm/helmfile.yaml --selector id=kcp
helmfile sync --file /training/50_setup-kdp-in-platform-cluster/helm/helmfile.yaml --selector id=developer-platform
```

There is no test suite, lint step, or build pipeline — `make verify` is the closest analogue to a smoke test.

## Lab architecture (big picture)

The labs build up two GCE Kubernetes clusters and layer kcp/KDP on top:

- **`00_prerequisites`** — ssh key, gcloud auth, `.trainingrc` setup.
- **`01_install-kcp-locally` → `04_sharing-apis`** — concept-only labs that run kcp as a local binary. They teach the kcp primitives (`kubectl ws`, workspaces, `APIResourceSchema`, `APIExport`, `APIBinding`) before any real cluster work. The local kcp data dir is `/training/.kcp/`.
- **`10_create-platform-cluster`, `11_create-provider-cluster`** — provision two GCE clusters using Terraform (`tf_infra/terraform.tfvars` + `.tf` files copied from the kubeone examples directory) followed by `kubeone apply`. The makefiles also rewrite the resulting kubeconfig user/context names to `admin@k8s-platform` / `admin@k8s-provider`, deposit them into `/training/.secrets/kubeconfig-*.yaml`, and squash them.
- **`12_setup-kcp-in-platform-cluster`** — installs ingress-nginx, cert-manager, DEX (OIDC IdP), and the kcp helm chart on the platform cluster; configures Let's Encrypt + GCP DNS records under `$PLATFORM_DOMAIN` and `internal.$PLATFORM_DOMAIN` (kcp front-proxy).
- **`13/14/15_create-kcp-*-kubeconfig`** — manually craft kubeconfigs for the three kcp personas (root admin, provider, consumer) using the kcp front-proxy CA and either client certs or service-account tokens. Resulting contexts: `root@kcp`, `base@kcp`, `provider@kcp`, `consumer@kcp`.
- **`20/21_*` (provider) → `30/31_*` (consumer) → `40_verify` → `41_teardown`** — end-to-end demo of providing a service: create the `MyService` CRD on the provider cluster, install the kcp `api-syncagent` helm chart pointed at the provider workspace, publish a `PublishedResource`, then bind/consume from the consumer workspace and verify the synced object lands back on the provider cluster.
- **`50_setup-kdp-in-platform-cluster`** — installs the KDP helm charts (`developer-platform`, `developer-platform-dashboard`) on top of kcp.
- **`60_provide-a-service` → `70_consume-a-service` → `99_teardown`** — repeats the provide/consume flow but driven through the KDP dashboard. Trainees download kubeconfigs from the dashboard UI and drag them into `/training/.secrets/`.

Key cross-cutting pieces:

- The kcp api-syncagent (`api-syncagent` helm chart, configured in `*_syncagent-helmfile.yaml`) is the bridge between a provider's real Kubernetes cluster and a kcp workspace; its `apiExportName` value must match the `APIExport` name in the kcp provider workspace.
- DNS, TLS, and OIDC are coupled: ingress-nginx's LB IP is captured into `INGRESS_IP`, written into Google Cloud DNS for `$PLATFORM_DOMAIN` / `*.$PLATFORM_DOMAIN`, then DEX issues OIDC tokens at `https://login.$PLATFORM_DOMAIN`, and cert-manager + Let's Encrypt secure both.
- The kcp front-proxy is exposed separately as a `LoadBalancer`; its IP (`KCP_FRONT_PROXY_IP`) is mapped to `internal.$PLATFORM_DOMAIN` and is what every kcp kubeconfig in `13/14/15` points at (`https://internal.$PLATFORM_DOMAIN:8443`).

## Working with this repo

- When a lab references an env var, assume it is supplied by `.trainingrc` — do not invent default values or hardcode trainee-specific data into committed files.
- Files like `<lab>/myservice_syncagent-helmfile.yaml`, `<lab>/myservice_published-resource.yaml`, and the various `kubeconfig*.yaml` are templates that the lab steps mutate in place via `sed`/`yq`. Preserve the placeholder tokens (`<DOMAIN>`, `<FILL-IN-YOUR-GCE-PROJECT-ID>`, `<FILL-IN-CLUSTER-NAME>`, `<FILL-IN-YOUR-PASSWORD>`, `your-email@example.com`) when editing — the makefiles substitute them at runtime.
- `.99_todos/` is the trainer's scratch area (open issues, slide notes); the devcontainer hides it from VS Code's file tree, but it is still part of the repo. Don't treat it as canonical content.
- The two clusters' kubeone configs live at `platform-cluster/kubeone.yaml` and `provider-cluster/kubeone.yaml`; their Terraform state is materialized into `*/tf_infra/` only after `make prepare-tf-config` copies the kubeone-provided `.tf` files in.
