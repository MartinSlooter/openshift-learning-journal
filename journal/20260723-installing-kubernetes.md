# Installing

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
