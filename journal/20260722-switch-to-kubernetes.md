# Switching to Kubernetes first
Since kubernetes is basically the foundation for OpenShift, I decided to first start with Kubernetes.

# Running VMs
I want to have 3 vms with static ip's. For that i use libvirt and add the mac addresses of the vms with an ip to the config of the default network.
The command I use virsh net-update (https://wiki.libvirt.org/Networking.html#virsh-net-update). More info on networking with libvirt: https://wiki.libvirt.org/VirtualNetworking.html#virtual-networking
```bash
sudo virsh net-update default add ip-dhcp-host "<host mac='52:54:00:90:43:fa' name='fedora-master0' ip='192.168.122.190' />" --live --config
```
