# Provisioning Compute Resources

On your machine you'll need to setup the Kubernetes control plane and the worker nodes where containers are ultimately run.

## Hostnames

In this will help you setup a local cluster with a generic name. If you're going to want multiple machines in the cluster in the future, please use the [original tutorial](https://github.com/kelseyhightower/kubernetes-the-hard-way/)

## Adding `/etc/hosts` Entries To Your Machine

In this section you will append(add to the end of the text file) hostname entries to your `/etc/hosts` file on your machine

Append the DNS entries from `hosts` to `/etc/hosts` via editing in a terminal:

```bash
sudo nano /etc/hosts
```

```text
## Add these if not already there
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Add the following to the file
127.0.0.1 cluster.kubernetes.local kubernetes.local cluster
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
