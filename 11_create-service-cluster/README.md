
# Create Service Cluster

<!-- TODO
install kcp - not in image
play around with kcp on first lab
 -->

```bash
# create the cluster
make -C /training/03_create-service-cluster create-cluster


# # rename the kubeconfig contexts
# TODO MAYBE BETTER WITHOUT RENAMING IT
# kubectl config rename-context kubernetes-admin@${TRAINEE_NAME}-platform-cluster platform-cluster
# kubectl config rename-context kubernetes-admin@${TRAINEE_NAME}-service-cluster service-cluster

# verify your kubeconfig
kubectx

# switch to service-cluster
kubectx service-cluster

# verify service cluster is working
kubectl get nodes

# TODO do i need this on service cluster
# cd /training/service-cluster; kubeone config machinedeployments -m /training/service-cluster/kubeone.yaml -t /training/service-cluster/tf_infra > /training/service-cluster/md.yaml
# sed -i 's/cluster-api-autoscaler-node-group-max-size: "1"/cluster-api-autoscaler-node-group-max-size: "3"/g' /training/service-cluster/md.yaml
# kubectl apply -f /training/service-cluster/md.yaml
# kubectl -n kube-system wait md/${TRAINEE_NAME}-service-cluster-pool1 --for=jsonpath='{.status.readyReplicas}'=1 --timeout=300s
# kubectl wait --for=condition=Ready node --all --timeout=300s
```
