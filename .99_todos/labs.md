# google credentials

should be automated via make

echo "export GOOGLE_CREDENTIALS='$(cat /training/.secrets/gcloud-service-account.json)'" >> /root/.trainingrc

# install krew plugins pin down versions

kubectl krew index add kcp-dev https://github.com/kcp-dev/krew-index.git
kubectl krew install kcp-dev/kcp
kubectl krew install kcp-dev/ws
kubectl krew install kcp-dev/create-workspace

# TODO security concerns?

kubectl create clusterrolebinding provider:admin \
  --clusterrole=cluster-admin \
  --serviceaccount=default:provider

<!-- TODO PS1 is not working -->
<!-- TODO hint to apiresourceschema got created and bound into apiexport -->
<!-- TODO hint to apiendpointslice -->

# after recview

add kro
quay passwords
version upgrades?
