# Set Up The Dev Machine

This lab assumes you're running these commands on the machine that will host kubernetes.

You'll need tools to administrate your Kubernetes cluster from the ground up. efore we get started we need to install a few command line utilities and clone the Kubernetes The Hard Arch Way git repository, which contains some additional configuration files that will be used to configure various Kubernetes components throughout this tutorial.

Launch the terminal on your machine

### Install Command Line Utilities To Manage Kubernetes

```bash
  sudo pacman -S wget curl vim openssl git kubectl
```

### Sync Git Repository

Now it's time to download a copy of this setup which contains the configuration files and templates that will be used build your Kubernetes cluster from the ground up. Clone the Kubernetes The Hard Way git repository using the `git` command:

```bash
cd ~/Downloads
git https://github.com/Doty1154/kubernetes-the-hard-arch-way.git
cd kubernetes-the-hard-arch-way
```
This will be the working directory for the rest of the setup. If you ever get lost run the `pwd` command to verify you are in the right directory when running commands on your machine :

```bash
pwd
```
for example 
```text
/root/kubernetes-the-hard-way
```
### Install Prerequisite Container Related Dependencies

```bash
sudo pacman -S cri-o crun  crictl 
```
### Install Kubernetes packages

In this section you will install the packages for the various Kubernetes components.

```bash
sudo pacman -S kube-proxy kubelet kube-apiserver kube-controller-manager kube-proxy kube-scheduler kubelet kubeadm
```

You'll need to install etcd, aur has it in its repo. you can install the package with [paru](https://github.com/Morganamilo/paru),

```bash
paru -S etcd
```

At this point `kubectl` is installed and can be verified by running the `kubectl` command:

```bash
kubectl version --client
```

```text
Client Version: v1.32.3
```

At this point the machine has been set up with all the command line tools and utilities necessary to complete the labs in this tutorial.

Next: [Provisioning Compute Resources](03-compute-resources.md)
