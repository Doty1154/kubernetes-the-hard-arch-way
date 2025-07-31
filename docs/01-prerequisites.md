# Prerequisites

In this lab you will review the machine requirements necessary to follow this tutorial.

## Machines

This tutorial allows you run run a single kubernetes node (which shouldn't be used for production as production systems should have atleast 3 machines in place). The following table lists the four machines and their CPU, memory, and storage requirements.

| Name    | Description            | CPU | RAM   | Storage |
|---------|------------------------|-----|-------|---------|
| server  | Kubernetes server      | 1   | 2GB   | 20GB    |

How you provision the machines is up to you, the only requirement is that each machine meet the above system requirements including the machine specs.
Ensure that your system is up to date 
```bash
sudo pacman -Syu
```

You should see something similar to the following output:

```text
:: Synchronizing package databases...
 core is up to date
 extra is up to date
 multilib is up to date
:: Starting full system upgrade...
 there is nothing to do

```

Next: [setting-up-the-jumpbox](02-jumpbox.md)
