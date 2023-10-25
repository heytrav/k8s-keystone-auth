# K8S Keystone Auth

Helm chart for deploying k8s-keystone-auth webhook to a Kubernetes cluster


## Usage

[Helm](https://helm.sh) must be installed to use the charts.  Please refer to
Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

  helm repo add k8s-keystone-auth https://heytrav.github.io/helm-charts

If you had already added this repo earlier, run `helm repo update` to retrieve
the latest versions of the packages.  You can then run `helm search repo
<alias>` to see the charts.

To install the k8s-keystone-auth chart:

    helm install my-k8s-keystone-auth k8s-keystone-auth/k8s-keystone-auth

To uninstall the chart:

    helm delete my-k8s-keystone-auth
