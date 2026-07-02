Below is an example of how you could organize a fully automated OpenShift lab deployment. This is not a drop-in production solution—it omits secrets and environment-specific values—but it demonstrates a workflow that many teams use.

openshift-lab/
├── ansible/
│   ├── inventory.yml
│   ├── deploy-vms.yml
│   ├── wait-for-bootstrap.yml
│   └── configure-cluster.yml
├── cluster-config/
│   ├── install-config.yaml
│   └── agent-config.yaml
├── scripts/
│   ├── create-iso.sh
│   ├── deploy-vms.sh
│   ├── wait-for-cluster.sh
│   └── destroy.sh
├── iso/
├── downloads/
└── README.md
create-iso.sh

This script assumes:

openshift-install is already downloaded
oc is installed
your cluster-config directory contains install-config.yaml and agent-config.yaml
#!/usr/bin/env bash

set -euo pipefail

INSTALLER=./downloads/openshift-install
CONFIG=./cluster-config
OUTPUT=./iso

echo "Cleaning previous ISO..."
rm -rf "${OUTPUT}"
mkdir -p "${OUTPUT}"

echo "Generating Agent ISO..."

${INSTALLER} agent create image \
    --dir "${CONFIG}" \
    --log-level=info

mv "${CONFIG}"/*.iso "${OUTPUT}/"

echo
echo "ISO created:"
ls -lh "${OUTPUT}"
deploy-vms.sh

Instead of using virsh directly, let Ansible handle VM creation.

#!/usr/bin/env bash

set -e

ansible-playbook \
    ansible/deploy-vms.yml
Example Ansible inventory
all:
  vars:
    vm_memory: 32768
    vm_vcpus: 16
    vm_disk: 150
    iso: "../iso/agent.x86_64.iso"

  hosts:
    bootstrap:
    master01:
    master02:
    master03:
deploy-vms.yml

This example uses the community.libvirt collection.

---
- hosts: localhost
  gather_facts: false

  tasks:

    - name: Create VM disks
      command:
        cmd: >
          qemu-img create
          -f qcow2
          /var/lib/libvirt/images/{{ item }}.qcow2
          {{ vm_disk }}G
      loop: "{{ groups['all'] }}"

    - name: Create virtual machines
      community.libvirt.virt:

        name: "{{ item }}"
        state: running
        memory: "{{ vm_memory }}"
        vcpus: "{{ vm_vcpus }}"

        disks:
          - name: "{{ item }}"
            device: disk
            size: "{{ vm_disk }}"

          - device: cdrom
            source: "{{ iso }}"

        networks:
          - network: default

      loop: "{{ groups['all'] }}"
wait-for-cluster.sh

This script waits until the Kubernetes API becomes available and then waits for installation to complete.

#!/usr/bin/env bash

set -euo pipefail

export KUBECONFIG=cluster-config/auth/kubeconfig

INSTALLER=downloads/openshift-install

echo "Waiting for bootstrap..."

${INSTALLER} wait-for bootstrap-complete \
    --dir cluster-config \
    --log-level=info

echo
echo "Waiting for installation..."

${INSTALLER} wait-for install-complete \
    --dir cluster-config \
    --log-level=info

echo
echo "Cluster installed."

oc get nodes

oc get co
Post-install configuration

After the installer finishes, you can use Ansible to configure the cluster.

---
- hosts: localhost

  tasks:

    - name: Accept all CSRs
      shell: |
        oc get csr -o name | xargs -r oc adm certificate approve

    - name: Wait until all operators are Available
      shell: |
        oc wait co --all \
          --for=condition=Available=True \
          --timeout=60m
Running the entire deployment

Your workflow can then be as simple as:

./scripts/create-iso.sh
./scripts/deploy-vms.sh
./scripts/wait-for-cluster.sh

ansible-playbook ansible/configure-cluster.yml
