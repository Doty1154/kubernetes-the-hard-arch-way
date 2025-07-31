# Bootstrapping the Kubernetes Worker Components

In this lab you will bootstrap two Kubernetes worker nodes. The following components used in this section: [runc](https://github.com/opencontainers/runc), [container networking plugins](https://github.com/containernetworking/cni), [containerd](https://github.com/containerd/containerd), [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet), and [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).

## Provisioning a Kubernetes Worker Components

Create the installation directories:

```bash
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

### Configure CNI Networking

Configure the `br-netfilter` kernel module:

```bash
sudo modprobe br-netfilter
sudo sh -c "echo "br-netfilter" >> /etc/modules-load.d/modules.conf"
```

```bash
sudo sh -c "echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.d/kubernetes.conf"
sudo sh -c "echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.d/kubernetes.conf"
sudo sysctl -p /etc/sysctl.d/kubernetes.conf
```
### Configure the Kubelet

Copy the `kubelet-config.yaml` configuration file:

```bash
sudo cp -v ./configs/kubelet-config.yaml /var/lib/kubelet/
```
Configure kubelet service launch arguments:
```bash
sudo nano /etc/kubernetes/kubelet.env
```
Edit the last line with the following.
```text
KUBELET_ARGS= --config=/var/lib/kubelet/kubelet-config.yaml --kubeconfig=/var/lib/kubelet/kubeconfig --hostname-override=kubelet
```
### Configure the Kubernetes Proxy

```bash
sudo cp -v ./configs/kube-proxy-config.yaml /var/lib/kube-proxy/
```
Configure kube-proxy service launch arguments:
```bash
sudo nano /etc/kubernetes/kube-proxy.env
```
Edit the last line with the following.
```text
KUBE_PROXY_ARGS=--config=/var/lib/kube-proxy/kube-proxy-config.yaml
```

### Start the Worker Services

```bash
sudo systemctl daemon-reload
sudo systemctl enable cri-o kubelet kube-proxy --now
sudo journalctl -xef -u kubelet -u kube-proxy -u cri-o
```

Check if the services are running:

```bash
sudo systemctl status kubelet kube-proxy cri-o
```

Be sure to complete the steps in this section before moving on to the next section.

## Verification

Run the following commands to list the registered Kubernetes node:

```bash
kubectl get nodes  -kubeconfig ./configs/admin.kubeconfig
```

```
NAME     STATUS   ROLES    AGE    VERSION
kubelet   Ready    <none>   1m     v1.32.3
```
Next: [Configuring kubectl for Remote Access](10-configuring-kubectl.md)
