## Prepare

```
sudo mkdir -p /var/lib/kubernetes/
sudo mv ca.pem ca-key.pem apiserver-key.pem apiserver.pem  /var/lib/kubernetes/
```
## Unit

```
export INTERNAL_IP=<REPLACE PRIVATE IP>

cat > apiserver.service <<EOF
[Unit]
Description=apiserver
Documentation=https://kubernetes.io

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --advertise-address=${INTERNAL_IP} \\
  --kubelet-preferred-address-types=InternalIP \\
  --allow-privileged=true \\
  --apiserver-count=1 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/apiserver.pem \\
  --etcd-keyfile=/var/lib/kubernetes/apiserver-key.pem \\
  --etcd-servers=https://${INTERNAL_IP}:2379 \\
  --event-ttl=1h \\
  --insecure-bind-address=0.0.0.0 \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/apiserver.pem \\
  --kubelet-client-key=/var/lib/kubernetes/apiserver-key.pem \\
  --kubelet-https=true \\
  --runtime-config=rbac.authorization.k8s.io/v1alpha1 \\
  --service-account-key-file=/var/lib/kubernetes/ca-key.pem \\
  --service-cluster-ip-range=10.3.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-ca-file=/var/lib/kubernetes/ca.pem \\
  --tls-cert-file=/var/lib/kubernetes/apiserver.pem \\
  --tls-private-key-file=/var/lib/kubernetes/apiserver-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF


sudo mv apiserver.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable apiserver
sudo systemctl start apiserver
```
