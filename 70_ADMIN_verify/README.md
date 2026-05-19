# Verify

> IMPORTANT: You are now the platform administrator!

In this lab you will check that KDP created all the needed resources.

KDP creates a CRD called `service.core.kdp.k8c.io`. This is an abstraction of kcp's APIExport and APIResourceSchema objects.

```bash
# verify KDP service exists
KUBECONFIG=/training/.secrets/kubeconfig-kdp-root.yaml kubectl get service.core.kdp.k8c.io
```
