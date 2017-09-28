Testing purposes only, one etcd member will suffice

# Etcd binnaries

(at remote node)

```
export ETCD_VERSION=v3.2.6
wget https://github.com/coreos/etcd/releases/download/${ETCD_VERSION}/etcd-${ETCD_VERSION}-linux-amd64.tar.gz -P /tmp/
tar -xvf /tmp/etcd-${ETCD_VERSION}-linux-amd64.tar.gz -C /tmp/
sudo mv /tmp/etcd-${ETCD_VERSION}-linux-amd64/etcd /tmp/etcd-${ETCD_VERSION}-linux-amd64/etcdctl /usr/local/bin/
```

# Prepare files and directories
```
sudo mkdir -p /etc/etcd /var/lib/etcd
sudo cp ca.pem apiserver-key.pem apiserver.pem /etc/etcd/
```

# Configuration
Create directories
```
sudo mkdir -p /etc/etcd /var/lib/etcd
```

Create unit file
```
export PRIVATE_IP=<REPLACE PRIVATE IP>
cat > etcd.service <<EOF
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name kube-etcd \\
  --cert-file=/etc/etcd/apiserver.pem \\
  --key-file=/etc/etcd/apiserver-key.pem \\
  --peer-cert-file=/etc/etcd/apiserver.pem \\
  --peer-key-file=/etc/etcd/apiserver-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${PRIVATE_IP}:2380 \\
  --listen-peer-urls https://${PRIVATE_IP}:2380 \\
  --listen-client-urls https://${PRIVATE_IP}:2379,http://127.0.0.1:2379 \\
  --advertise-client-urls https://${PRIVATE_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster kube-etcd=https://${PRIVATE_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF


sudo mv etcd.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```
