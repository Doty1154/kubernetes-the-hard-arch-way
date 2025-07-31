# Generating Kubernetes Configuration Files for Authentication

In this lab you will generate [Kubernetes client configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), typically called kubeconfigs, which configure Kubernetes clients to connect and authenticate to Kubernetes API Servers.

## Client Authentication Configs

In this section you will generate kubeconfig files for the `kubelet` and the `admin` user.

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/).

> The following commands must be run in the same directory used to generate the SSL certificates during the [Generating TLS Certificates](04-certificate-authority.md) lab.

Generate a kubeconfig file for worker kubernetes components:

```bash
  cd ~/kubernetes-the-hard-arch-way
  mkdir kubeconfigs

  kubectl config set-cluster devcluster \
    --certificate-authority=./certs/ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=./kubeconfigs/kubelet.kubeconfig

  kubectl config set-credentials system:node:kubelet \
    --client-certificate=./certs/kubelet.crt \
    --client-key=./certs/kubelet.key \
    --embed-certs=true \
    --kubeconfig=./kubeconfigs/kubelet.kubeconfig

  kubectl config set-context default \
    --cluster=devcluster \
    --user=system:node:kubelet \
    --kubeconfig=./kubeconfigs/kubelet.kubeconfig

  kubectl config use-context default \
    --kubeconfig=./kubeconfigs/kubelet.kubeconfig

```

### The kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `kube-proxy` service:

```bash

  kubectl config set-cluster devcluster \
    --certificate-authority=./certs/ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=./kubeconfigs/kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=./certs/kube-proxy.crt \
    --client-key=./certs/kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=./kubeconfigs/kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=devcluster \
    --user=system:kube-proxy \
    --kubeconfig=./kubeconfigs/kube-proxy.kubeconfig

  kubectl config use-context default \
    --kubeconfig=./kubeconfigs/kube-proxy.kubeconfig

```

### The kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `kube-controller-manager` service:

```bash

  kubectl config set-cluster devcluster \
    --certificate-authority=./certs/ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=./kubeconfigs/kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=./certs/kube-controller-manager.crt \
    --client-key=./certs/kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=./kubeconfigs/kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=devcluster \
    --user=system:kube-controller-manager \
    --kubeconfig=./kubeconfigs/kube-controller-manager.kubeconfig

  kubectl config use-context default \
    --kubeconfig=./kubeconfigs/kube-controller-manager.kubeconfig

```


### The kube-scheduler Kubernetes Configuration File

Generate a kubeconfig file for the `kube-scheduler` service:

```bash

  kubectl config set-cluster devcluster \
    --certificate-authority=./certs/ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=./kubeconfigs/kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=./certs/kube-scheduler.crt \
    --client-key=./certs/kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=./kubeconfigs/kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=devcluster \
    --user=system:kube-scheduler \
    --kubeconfig=./kubeconfigs/kube-scheduler.kubeconfig

  kubectl config use-context default \
    --kubeconfig=./kubeconfigs/kube-scheduler.kubeconfig

```


### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```bash

  kubectl config set-cluster devcluster \
    --certificate-authority=./certs/ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=./kubeconfigs/admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=./certs/admin.crt \
    --client-key=./certs/admin.key \
    --embed-certs=true \
    --kubeconfig=./kubeconfigs/admin.kubeconfig

  kubectl config set-context default \
    --cluster=devcluster \
    --user=admin \
    --kubeconfig=./kubeconfigs/admin.kubeconfig

  kubectl config use-context default \
    --kubeconfig=./kubeconfigs/admin.kubeconfig

```

## Distribute the Kubernetes Configuration Files

Copy the `kubelet` and `kube-proxy` kubeconfig files for the services.

```bash

  sudo mkdir -p /var/lib/{kube-proxy,kubelet}

  sudo cp -v ./kubeconfigs/kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig 

  sudo cp -v ./kubeconfigs/kubelet.kubeconfig /var/lib/kubelet/kubeconfig

```

Next: [Generating the Data Encryption Config and Key](06-data-encryption-keys.md)
