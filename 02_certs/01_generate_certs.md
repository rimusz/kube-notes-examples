https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/


## Create output dir
```
mkdir  -p ./gen
```

## Customize Openssl config
For each kubelet and for apiserver, customize `*.conf` files.

## CA
```
openssl genrsa -out ./gen/ca-key.pem 2048
openssl req -x509 -new -nodes -key ./gen/ca-key.pem -days 3650 -out ./gen/ca.pem -subj "/CN=kubernetes"
```

# API Server
```
openssl genrsa -out ./gen/apiserver-key.pem 2048
openssl req -new -key ./gen/apiserver-key.pem -out ./gen/apiserver.csr -subj "/CN=apiserver" -config apiserver.conf
openssl x509 -req -in ./gen/apiserver.csr -CA ./gen/ca.pem -CAkey ./gen/ca-key.pem -CAcreateserial -out ./gen/apiserver.pem -days 3650 -extensions v3_req -extfile apiserver.conf
```

# Admin
```
openssl genrsa -out ./gen/admin-key.pem 2048
openssl req -new -key ./gen/admin-key.pem -out ./gen/admin.csr -subj "/CN=admin/O=system:masters"
openssl x509 -req -in ./gen/admin.csr -CA ./gen/ca.pem -CAkey ./gen/ca-key.pem -CAcreateserial -out ./gen/admin.pem -days 3650
```

# Kubelet
TODO loop, create a script that reads a config file containing each node info
```
openssl genrsa -out ./gen/kube-01-key.pem 2048
openssl req -new -key ./gen/kube-01-key.pem -out ./gen/kube-01.csr -subj "/CN=kube-01/O=system:nodes" -config kube-01.conf
openssl x509 -req -in ./gen/kube-01.csr -CA ./gen/ca.pem -CAkey ./gen/ca-key.pem -CAcreateserial -out ./gen/kube-01.pem -days 3650 -extensions v3_req -extfile kube-01.conf

openssl genrsa -out ./gen/kube-02-key.pem 2048
openssl req -new -key ./gen/kube-02-key.pem -out ./gen/kube-02.csr -subj "/CN=kube-02/O=system:nodes" -config kube-02.conf
openssl x509 -req -in ./gen/kube-02.csr -CA ./gen/ca.pem -CAkey ./gen/ca-key.pem -CAcreateserial -out ./gen/kube-02.pem -days 3650 -extensions v3_req -extfile kube-02.conf

openssl genrsa -out ./gen/kube-03-key.pem 2048
openssl req -new -key ./gen/kube-03-key.pem -out ./gen/kube-03.csr -subj "/CN=kube-03/O=system:nodes" -config kube-03.conf
openssl x509 -req -in ./gen/kube-03.csr -CA ./gen/ca.pem -CAkey ./gen/ca-key.pem -CAcreateserial -out ./gen/kube-03.pem -days 3650 -extensions v3_req -extfile kube-03.conf
```

# Kube Proxy
```
openssl genrsa -out ./gen/proxy-key.pem 2048
openssl req -new -key ./gen/proxy-key.pem -out ./gen/proxy.csr -subj "/CN=proxy/O=system:node-proxier"
openssl x509 -req -in ./gen/proxy.csr -CA ./gen/ca.pem -CAkey ./gen/ca-key.pem -CAcreateserial -out ./gen/proxy.pem -days 3650
```

# (Optional) check certs
```
openssl x509 -in $CERT_FILE -text -noout
```
