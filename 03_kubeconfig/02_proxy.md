## Execute for every node

```
export APISERVER_ADDRESS=10.132.109.203
export CERTS_PATH=../02_certs/gen

mkdir -p ./gen

kubectl config set-cluster hardway \
  --certificate-authority ${CERTS_PATH}/ca.pem \
  --embed-certs=true \
  --server=https://${APISERVER_ADDRESS}:6443 \
  --kubeconfig=./gen/proxy.kubeconfig

kubectl config set-credentials system:node-proxier \
  --client-certificate=${CERTS_PATH}/proxy.pem \
  --client-key=${CERTS_PATH}/proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=./gen/proxy.kubeconfig

kubectl config set-context default \
  --cluster=hardway \
  --user=system:node-proxier \
  --kubeconfig=./gen/proxy.kubeconfig

kubectl config use-context default --kubeconfig=./gen/proxy.kubeconfig

cat ./gen/proxy.kubeconfig
```
