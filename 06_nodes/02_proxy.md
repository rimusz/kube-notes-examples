## Move  assets to proxy dir
```
mv proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

## Create Unit
```
cat > proxy.service <<EOF
[Unit]
Description=proxy
Documentation=https://kubernetes.io

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --cluster-cidr=10.2.0.0/16 \\
  --kubeconfig=/var/lib/kube-proxy/kubeconfig \\
  --proxy-mode=iptables \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF


sudo mv proxy.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable proxy
sudo systemctl start proxy
```
