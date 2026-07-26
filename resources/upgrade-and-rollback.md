**Here’s a complete, practical, and safe upgrade + rollback procedure.**

This is the method most experienced people use in production.

---

### Phase 1: Preparation (Before any upgrade)

#### 1. Take an etcd snapshot (Most important step)

```bash
# Create a snapshot directory
sudo mkdir -p /root/etcd-backups

# Take the snapshot
sudo ETCDCTL_API=3 etcdctl snapshot save /root/etcd-backups/etcd-snapshot-$(date +%Y%m%d-%H%M).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

Verify the snapshot:

```bash
sudo ETCDCTL_API=3 etcdctl snapshot status /root/etcd-backups/etcd-snapshot-*.db -w table
```

#### 2. Backup important files

```bash
sudo tar -czvf /root/k8s-backup-$(date +%Y%m%d).tar.gz \
  /etc/kubernetes \
  /var/lib/etcd \
  /root/etcd-backups
```

---

### Phase 2: Safe Upgrade Procedure

```bash
# 1. Update the package repository to the new version (example: 1.35 → 1.36)
# (Fedora example)
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.36/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.36/rpm/repodata/repomd.xml.key
EOF

# 2. Upgrade kubeadm first
sudo dnf install -y kubeadm-1.36.*

# 3. Check what the upgrade will do
sudo kubeadm upgrade plan

# 4. Perform the control-plane upgrade
sudo kubeadm upgrade apply v1.36.x -y

# 5. Upgrade kubelet and kubectl
sudo dnf install -y kubelet-1.36.* kubectl-1.36.*
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# 6. Verify
kubectl get nodes
kubectl get pods -A
```

---

### Phase 3: Rollback Procedure (If something breaks)

#### Option A: Fast rollback (when upgrade failed early)

```bash
# Change repository back to previous version
# Then reinstall previous packages
sudo dnf install -y kubeadm-1.35.* kubelet-1.35.* kubectl-1.35.*

sudo kubeadm upgrade apply v1.35.x --force
sudo systemctl restart kubelet
```

#### Option B: Full rollback using etcd snapshot (Recommended when things are broken)

```bash
# 1. Stop kubelet
sudo systemctl stop kubelet

# 2. Move current etcd data (safety)
sudo mv /var/lib/etcd /var/lib/etcd.broken

# 3. Restore the snapshot
sudo ETCDCTL_API=3 etcdctl snapshot restore /root/etcd-backups/etcd-snapshot-YYYYMMDD-HHMM.db \
  --data-dir=/var/lib/etcd

# 4. Fix ownership
sudo chown -R etcd:etcd /var/lib/etcd

# 5. Reinstall the previous version packages
sudo dnf install -y kubeadm-1.35.* kubelet-1.35.* kubectl-1.35.*

# 6. Restart everything
sudo systemctl start kubelet

# 7. Verify
kubectl get nodes
kubectl get pods -A
```

---

### Extra Tips

- Always upgrade one minor version at a time (1.35 → 1.36 is good, skipping versions is riskier).
- On multi-node clusters you must also upgrade worker nodes one by one (drain → upgrade → uncordon).
- Keep at least the last 2–3 etcd snapshots.

---
