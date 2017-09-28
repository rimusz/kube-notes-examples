## Execute for every node

```
export NODE=kube-01
export APISERVER_ADDRESS=10.132.109.203
export CERTS_PATH=../02_certs/gen

mkdir -p ./gen

kubectl config set-cluster hardway \
  --certificate-authority ${CERTS_PATH}/ca.pem \
  --embed-certs=true \
  --server=https://${APISERVER_ADDRESS}:6443 \
  --kubeconfig=./gen/${NODE}.kubeconfig

kubectl config set-credentials system:node:${NODE} \
  --client-certificate=${CERTS_PATH}/${NODE}.pem \
  --client-key=${CERTS_PATH}/${NODE}-key.pem \
  --embed-certs=true \
  --kubeconfig=./gen/${NODE}.kubeconfig

kubectl config set-context default \
  --cluster=hardway \
  --user=system:node:${NODE} \
  --kubeconfig=./gen/${NODE}.kubeconfig

kubectl config use-context default --kubeconfig=./gen/${NODE}.kubeconfig

cat ./gen/${NODE}.kubeconfig
```
