# OpenShift Local

## Create VM
First try I created a VM with only 8G. This will only allow MicroShift mode, which is not the outcome I wanted.
Recreate VM with 12G so the crc installer can fully install.

```bash
cd ~/Downloads/

tar xvf crc-linux-amd64.tar.xz 

mkdir -p ~/local/bin

mv crc-linux-*-amd64/crc ~/local/bin/

export PATH=$HOME/local/bin:$PATH

crc version

echo 'export PATH=$HOME/local/bin:$PATH' >> ~/.bashrc 
```

```bash
crc config set consent-telemetry no

crc config view

crc setup
```

## Starting OpenShift Local
```bash
crc start -p ~/Downloads/pull-secret
```


