## create local kubeconfig


```
export APISERVER_ADDRESS=<EXTERNAL ADDRESS>
export CERTS_PATH=../02_certs/gen

kubectl config set-cluster hardway \
  --certificate-authority ${CERTS_PATH}/ca.pem \
  --embed-certs=true \
  --server=https://${APISERVER_ADDRESS}:6443

kubectl config set-credentials admin \
  --client-certificate=${CERTS_PATH}/admin.pem \
  --client-key=${CERTS_PATH}/admin-key.pem \
  --embed-certs=true

kubectl config set-context default \
  --cluster=hardway \
  --user=admin

kubectl config use-context default

cat ~/.kube/config
```
