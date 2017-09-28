
## Move  assets to kubelet/kubernetes dir
```
export NODE=kube-01

mv ${NODE}.pem ${NODE}-key.pem /var/lib/kubelet/
mv ca.pem /var/lib/kubernetes/

```

## Create Unit

```
export NODE=kube-01
cat > kubelet.service <<EOF
[Unit]
Description=kubelet
Documentation=https://kubernetes.io

[Service]
ExecStart=/usr/local/bin/kubelet \\
--allow-privileged=true \\
--cluster-dns=10.3.0.10 \\
--cluster-domain=cluster.local \\
--image-pull-progress-deadline=2m \\
--kubeconfig=/var/lib/kubelet/kubeconfig \\
--network-plugin=cni \\
--pod-cidr=10.2.0.0/16 \\
--register-node=true \\
--require-kubeconfig \\
--runtime-request-timeout=10m \\
--tls-cert-file=/var/lib/kubelet/${NODE}.pem \\
--tls-private-key-file=/var/lib/kubelet/${NODE}-key.pem \\
--v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF


sudo mv kubelet.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable kubelet
sudo systemctl start kubelet
```
