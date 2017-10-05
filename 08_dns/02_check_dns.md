## Check kube DNS

https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

Create busybox pod:

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
EOF

```

Check DNS:

```
kubectl exec busybox -- nslookup kubernetes
kubectl exec busybox -- nslookup service_name.default.svc.cluster.local
kubectl exec busybox -- nslookup 10-2-1-27.default.pod.cluster.local
kubectl exec busybox -- nslookup pod_name.etcd-cluster.default.svc
```
