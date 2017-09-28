
# Upload kubeconfig

```
export NODE_IP=<EXTERNAL-IP>
export NODE=<NODE>
scp -i ${SSH_KEY} ./gen/${NODE}.kubeconfig ./gen/proxy.kubeconfig root@${NODE_IP}:~/
```
