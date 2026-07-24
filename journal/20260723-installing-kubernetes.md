# Installing Kubernetes on Fedora

## Goal
Is to understand understanding the installation.

## What I learned
- Troubleshooting the installation;
- Unfortunately solutions did not work;
- Starting over with same distro as guide I am following.

## Progress
Installed the cluster software. However initializing the cluster failed.

## Installing

## k8s components
Then I install all kubernetes components.
```bash
sudo dnf install kubelet kubeadm kubectl

sudo dnf install dnf-plugins-core
sudo dnf versionlock add kubelet kubeadm kubectl
sudo dnf versionlock list
```

## Rerun init
I seems like the master vm was sized too small. Sized RAM to 8GB and try init again.
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
