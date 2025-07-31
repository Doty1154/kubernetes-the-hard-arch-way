### Install Prerequisites 
Install helm and helmfile, these are one of the package managers for kubernetes available in the ecosystem.

```bash
sudo pacman -S helm helmfile
```
```bash
mkdir -p ./core-services/flannel
nano flannel-install.yaml
```
### Install Flannel
Install instructions based on flannel docs
https://github.com/flannel-io/flannel

You should make sure you're installing the latest release by updating the version string to the most recent app version posted on [flannels git repo](https://github.com/flannel-io/flannel/blob/master/chart/kube-flannel/Chart.yaml)
modify flannel-install.yaml with the following:
```text
repositories:
- name: flannel
  url: https://flannel-io.github.io/flannel

releases:
- name: flannel
  namespace: kube-flannel
  chart:   flannel/flannel
  version: v0.27.2
  installed: true
  values:
    - flannel-values.yaml
```
Download the softwares settings
```bash
wget https://raw.githubusercontent.com/flannel-io/flannel/refs/heads/master/chart/kube-flannel/values.yaml --output-document ./core-services/flannel/flannel-values.yaml
```
Modify the following in the flannel-values.yaml
```bash
nano ./core-services/flannel/flannel-values.yaml
```
```text
  enableNFTables: false ->   enableNFTables: true
  backend: "vxlan" -> backend: "host-gw"
```
This command will trigger helm to install the selected version with the selected settings.
```bash
helmfile --file ./core-services/flannel/flannel-install.yaml sync --wait --wait-for-jobs
```
