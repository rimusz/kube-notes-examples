## Unit

```
export INTERNAL_IP=<REPLACE PRIVATE IP>

cat > scheduler.service <<EOF
[Unit]
Description=scheduler
Documentation=https://kubernetes.io

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --leader-elect=true \\
  --master=http://${INTERNAL_IP}:8080 \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo mv scheduler.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable scheduler
sudo systemctl start scheduler
```
