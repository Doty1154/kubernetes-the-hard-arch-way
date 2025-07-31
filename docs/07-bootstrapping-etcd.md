# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). In this lab you will bootstrap a single node etcd cluster.

### Configure the etcd Server
Create etcd.env to contain settings needed for kubernetes by editing it with sudo nano. 

```bash
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo nano /etc/etcd/etcd.env

```

Paste in the following.
```text
# Kubernetes etcd arguments
#
# The ETCD_ARGS environment variable is used to provide flags and options to
# etcd when running etcd.service.
# See `etcd --help` for further information.
#
ETCD_ARGS= --name controller --initial-advertise-peer-urls http://127.0.0.1:2380 --listen-peer-urls http://127.0.0.1:2380 --listen-client-urls http://127.0.0.1:2379 --advertise-client-urls http://127.0.0.1:2379 --initial-cluster-token etcd-cluster-0 --initial-cluster controller=http://127.0.0.1:2380 --initial-cluster-state new --data-dir=/var/lib/etcd

```
Then modify a systemd override service conf to contain the launch settings.


```bash
  sudo mkdir /etc/systemd/system/etcd.service.d
  sudo nano /etc/systemd/system/etcd.service.d/override.conf

```
Copy paste these settings in
```text
 [Service]
 ExecStart=
 ExecStart=-/usr/bin/etcd $ETCD_ARGS
 EnvironmentFile=-/etc/etcd/etcd.env
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:


### Copy The Certificates And Start the etcd Server

```bash
  sudo cp -v ./certs/ca.crt ./certs/kube-api-server.key ./certs/kube-api-server.crt /etc/etcd/
  sudo systemctl daemon-reload
  sudo systemctl enable etcd --now

```

## Verification

List the etcd cluster members:

```bash
etcdctl member list
```

```text
6702b0a34e2cfd39, started, controller, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

Next: [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)
