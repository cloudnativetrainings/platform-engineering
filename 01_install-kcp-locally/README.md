
# Install kcp locally

In this lab you will install

* kcp locally
* add the kubectl plugin manager called `krew`
* install some krew plugins needed for working with kcp

## Install kcp

```bash
# install kcp
KCP_VERSION=0.33.6 \
  && curl -fsSLO "https://github.com/kcp-dev/kcp/releases/download/v${KCP_VERSION}/kcp_${KCP_VERSION}_linux_amd64.tar.gz" \
  && mkdir -p /training/kcp_${KCP_VERSION}_linux_amd64 \
  && tar zxvf /training/kcp_${KCP_VERSION}_linux_amd64.tar.gz -C /training/kcp_${KCP_VERSION}_linux_amd64 \
  && cp /training/kcp_${KCP_VERSION}_linux_amd64/bin/kcp /usr/local/bin/ \
  && echo "export KCP_VERSION=${KCP_VERSION}" | tee -a /root/.trainingrc

# verify
kcp --version
```

## Install krew plugins

```bash
# install krew plugins
kubectl krew index add kcp-dev https://github.com/kcp-dev/krew-index.git
kubectl krew install kcp-dev/kcp
kubectl krew install kcp-dev/ws
kubectl krew install kcp-dev/create-workspace

# verify
kubectl krew list
```
