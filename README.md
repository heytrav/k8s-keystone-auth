# K8S Keystone Auth

Deploy `k8s-keystone-auth` on a Kubernetes cluster to allow RBAC with Openstack.

## Prerequisites

The following must be set up in the **control plane** of a kubernetes cluster:

* `kube-apiserver` must be configured to allow **Webhook** authorization and have the appropriate webhook config in place

   ```yaml
   command:
         - kube-apiserver
         .
         .
         - --authorization-mode=Node,Webhook,RBAC                                                                                                                                                         │
         - --authentication-token-webhook-config-file=/etc/kubernetes/webhooks/keystone_webhook_config.yaml                                                                                               │
         - --authorization-webhook-config-file=/etc/kubernetes/webhooks/keystone_webhook_config.yaml    
         .
   ```
* Webhook config (i.e. `keystone_webhook_config.yaml`) deployed on the control plane

   ```yaml
   ---
   apiVersion: v1
   kind: Config
   preferences: {}
   clusters:
     - cluster:
         insecure-skip-tls-verify: true
         server: https://127.0.0.1:8443/webhook
       name: webhook
   users:
     - name: webhook
   contexts:
     - context:
         cluster: webhook
         user: webhook
       name: webhook
   current-context: webhook
   ```


With the webhook configuration in place and `kube-apiserver` configured as above, 
the apiserver will direct RBAC authentication/authorization requests to an https
endpoint as specified by `clusters[0].cluster.server`.

## Installing RBAC

To install k8s keystone auth run the following:

```
helm  upgrade k8s-keystone-auth --install $THISHELMCHART --version $HELMCHARTVERSION --set openstackAuthUrl=$OS_AUTH_URL --set projectId=$OS_PROJECT_ID
```

## Authenticating

To interact with a cluster using RBAC you'll need the following:

* A valid commandline session (i.e using your OpenStack rc file)
* Kubeconfig file set up for RBAC

In your `kubeconfig` file:

```yaml

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: *************
    server: https://###.###.###.###:6443
  name: myclustername
contexts:
- context:
    cluster: myclustername
    user: myclustername-admin
  name: myusername@myclustername
current-context: myusername@myclustername
kind: Config
preferences: {}
users:
- name: myusername
  user:
    exec:
      command: /bin/bash
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
        - -c
        - >
          if [ -z ${OS_TOKEN} ]; then
              echo 'Error: Missing OpenStack credential from environment variable $OS_TOKEN' > /dev/stderr
              exit 1
          else
              echo '{ "apiVersion": "client.authentication.k8s.io/v1beta1", "kind": "ExecCredential", "status": { "token": "'"${OS_TOKEN}"'"}}'
          fi




```

**Note:** For clusters created using Magnum, the `openstack` magnum client should generate the appropriate `kubeconfig` file for you with the command:

```
openstack coe cluster config $CLUSTERNAME --use-keystone
```
