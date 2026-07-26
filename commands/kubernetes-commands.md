# Commands

## Administration
Join nodes to the cluster
```bash
kubeadm token create --print-join-command
```

Check for versions in repo's
```bash
# Check a few recent versions
for v in 1.30 1.31 1.32 1.33 1.34 1.35 1.36 1.37; do
  echo -n "v$v: "
  curl -s -o /dev/null -w "%{http_code}" https://pkgs.k8s.io/core:/stable:/v$v/rpm/ && echo " available" || echo " not found"
done
```
