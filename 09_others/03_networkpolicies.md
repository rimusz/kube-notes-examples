## Create Network Policy

https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/
https://ahmet.im/blog/kubernetes-network-policy/

Network policy needs
- podSelector (empty means all namespace)
- ingress (whitelist rules)
  - from (namespaceSelector, podSelector)
  - port

Create test namespaces
```
kubectl create ns isolated
kubectl create ns public
```

Isolate namespace
```
kubectl annotate ns isolated "net.beta.kubernetes.io/network-policy={\"ingress\":{\"isolation\":\"DefaultDeny\"}}"
```

Deploy nginx-isolated at isolated
```
kubectl run nginx-isolated --namespace isolated --image=nginx
kubectl expose deployment --namespace isolated nginx-isolated --port 80
```

Deploy nginx-public at public
```
kubectl run nginx-public --namespace public --image=nginx
kubectl expose deployment --namespace public nginx-public --port 80
```


Test isolation
```
kubectl run bb --rm -ti --image=busybox /bin/sh

wget -q --timeout=5  nginx-public.public -O -
<should show nginx default index page>

wget -q --timeout=5  nginx-isolated.isolated -O -

```

# Others

Isolate a namespace
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
  name: isolated-default-deny
  namespace: isolated
spec:
  podSelector:
EOF
```

Allow access to isolated nginx
```
kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: extensions/v1beta1
metadata:
  name: isolated-nginx
  namespace: isolated
spec:
  podSelector:
    matchLabels:
      run: nginx-isolated
  ingress:
    - from:
      - podSelector:
          matchLabels: {}
EOF
```

Allow access to isolated nginx from namespace client
```
kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: extensions/v1beta1
metadata:
  name: isolated-nginx
  namespace: isolated
spec:
  podSelector:
    matchLabels:
      run: nginx-isolated
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            layer: client
EOF
```

```
kind: NetworkPolicy
apiVersion: extensions/v1beta1
metadata:
  namespace: isolated
  name: isolated-nginx-policy
spec:
  podSelector:
    matchLabels:
      run: isolated-nginx
  ingress:
    - from:
        - podSelector:
            matchLabels:
              run: busybox
      ports:
        - protocol: TCP
          port: 80
```
