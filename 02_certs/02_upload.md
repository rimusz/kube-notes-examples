# Control plane

```
export MASTER_IP=<REPLACE>
scp -i ${SSH_KEY} ./gen/ca.pem ./gen/ca-key.pem ./gen/apiserver.pem ./gen/apiserver-key.pem root@${MASTER_IP}:~/
```

# Workers

```
export NODE_IP=<REPLACES>
export NODE=kube-01
scp -i ${SSH_KEY} ./gen/ca.pem ./gen/ca-key.pem ./gen/proxy.pem ./gen/proxy-key.pem ./gen/${NODE}.pem ./gen/${NODE}-key.pem root@${NODE_IP}:~/
```
