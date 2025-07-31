# Bootstrapping the Kubernetes Control Plane

This will configure the Kubernetes control plane. The following components will configured on your machine: Kubernetes API Server, Scheduler, and Controller Manager.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```bash
sudo mkdir -p /etc/kubernetes/config
```

### Configure the Kubernetes API Server

```bash
  cd ~/kubernetes-the-hard-arch-way
  sudo mkdir -p /var/lib/kubernetes/

  sudo cp -v ./certs/ca.crt /var/lib/kubernetes/
  sudo cp -v ./certs/ca.key /var/lib/kubernetes/
  sudo cp -v ./certs/kube-api-server.key /var/lib/kubernetes/
  sudo cp -v ./certs/kube-api-server.crt /var/lib/kubernetes/
  sudo cp -v ./certs/service-accounts.key /var/lib/kubernetes/
  sudo cp -v ./certs/service-accounts.crt /var/lib/kubernetes/
  sudo cp -v ./certs/kube-api-server.key /var/lib/kubernetes/
  sudo cp -v ./certs/encryption-config.yaml /var/lib/kubernetes/

```

Configure the `kube-apiserver` service launch arguments([This is based on the original source systemd unit](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/units/kube-apiserver.service)):

```bash
sudo nano /etc/kubernetes/kube-apiserver.env

```
Edit the last line with the following. You may want to edit `--service-node-port-range=30000-32767` to a lower starting port if you want pods to use publicly accessible node ports(this isn't a good practice for production but you may want to do this in testing.) Something like `--service-node-port-range=51-32767` would allow you to host dns services accessibly via node port.
```text
KUBE_APISERVER_ARGS= --allow-privileged=true --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/var/log/audit.log --authorization-mode=Node,RBAC --bind-address=0.0.0.0 --client-ca-file=/var/lib/kubernetes/ca.crt --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota --etcd-servers=http://127.0.0.1:2379 --event-ttl=1h --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt --kubelet-client-certificate=/var/lib/kubernetes/kube-api-server.crt --kubelet-client-key=/var/lib/kubernetes/kube-api-server.key --runtime-config='api/all=true' --service-account-key-file=/var/lib/kubernetes/service-accounts.crt --service-account-signing-key-file=/var/lib/kubernetes/service-accounts.key --service-account-issuer=https://server.kubernetes.local:6443 --service-node-port-range=30000-32767 --tls-cert-file=/var/lib/kubernetes/kube-api-server.crt --tls-private-key-file=/var/lib/kubernetes/kube-api-server.key

```
You'll want to create the audit log and set permissions

```bash
sudo sh -c "echo "" > /var/log/audit.log"
sudo chown kube:kube /var/log/audit.log
```
### Configure the Kubernetes Controller Manager

Copy the `kube-controller-manager` kubeconfig into place:

```bash
sudo cp -v ./kubeconfigs/kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Configure the `kube-controller-manager.service` service launch arguments ([This is based on the original source systemd unit](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/units/kube-controller-manager.service)):

```bash
sudo nano /etc/kubernetes/kube-apiserver.env

```
Edit the last line with the following.
```text
KUBE_CONTROLLER_MANAGER_ARGS=--bind-address=0.0.0.0 --cluster-cidr=10.200.0.0/16 --cluster-name=cluster --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt --cluster-signing-key-file=/var/lib/kubernetes/ca.key --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig --root-ca-file=/var/lib/kubernetes/ca.crt --service-account-private-key-file=/var/lib/kubernetes/service-accounts.key --service-cluster-ip-range=10.32.0.0/24 --use-service-account-credentials=true
```

### Configure the Kubernetes Scheduler

Copy the `kube-scheduler` kubeconfig into place:

```bash
sudo cp -v ./kubeconfigs/kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Copy the `kube-scheduler.yaml` configuration file:

```bash
sudo cp -v ./configs/kube-scheduler.yaml /etc/kubernetes/config/
```

Configure the `kube-scheduler` service launch arguments([This is based on the original source systemd unit](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/units/kube-controller-manager.service))::

```bash
sudo nano /etc/kubernetes/kube-scheduler.env
```
Edit the last line with the following.
```text
KUBE_SCHEDULER_ARGS=--config=/etc/kubernetes/config/kube-scheduler.yaml      

```

### Start the Controller Services

```bash
  sudo systemctl daemon-reload

  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler --now
```
You may want to read the logs from the services as they launch in case there's any errors launching. 
```bash
sudo journalctl -xef -u kube-apiserver -u kube-controller-manager -u kube-scheduler -u etcd
```
> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.

You can check if any of the control plane components are happy and healthy by viewing the status with systemctl

```bash
sudo systemctl status kube-apiserver kube-controller-manager kube-scheduler
```
You should see "Active: active (running)" next to each service name.

### Verification

At this point the Kubernetes control plane components should be up and running. Verify this using the `kubectl` command line tool:

```bash
kubectl cluster-info \
  --kubeconfig ./kubeconfigs/admin.kubeconfig
```

```text
Kubernetes control plane is running at https://127.0.0.1:6443
```

## RBAC for Kubelet Authorization

In this section you will configure RBAC permissions to allow the Kubernetes API Server to access the Kubelet API on each worker node. Access to the Kubelet API is required for retrieving metrics, logs, and executing commands in pods.

> This tutorial sets the Kubelet `--authorization-mode` flag to `Webhook`. Webhook mode uses the [SubjectAccessReview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#checking-api-access) API to determine authorization.

The commands in this section will affect the entire cluster.
Create the `system:kube-apiserver-to-kubelet` [ClusterRole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) with permissions to access the Kubelet API and perform most common tasks associated with managing pods:

```bash
kubectl apply -f ./configs/kube-apiserver-to-kubelet.yaml --kubeconfig ./kubeconfigs/admin.kubeconfig
```

### Verification

At this point the Kubernetes control plane is up and running. Run the following commands to verify it's working:

Make a HTTP request for the Kubernetes version info:

```bash
curl --cacert ca.crt \
  https://server.kubernetes.local:6443/version
```

```text
{
  "major": "1",
  "minor": "32",
  "gitVersion": "v1.32.3",
  "gitCommit": "32cc146f75aad04beaaa245a7157eb35063a9f99",
  "gitTreeState": "clean",
  "buildDate": "2025-03-11T19:52:21Z",
  "goVersion": "go1.23.6",
  "compiler": "gc",
  "platform": "linux/arm64"
}
```

Next: [Bootstrapping the Kubernetes Worker Nodes](09-bootstrapping-kubernetes-workers.md)
