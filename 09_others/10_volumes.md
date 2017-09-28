# Volumes

https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/
https://kubernetes.io/docs/concepts/storage/volumes/#emptydir

Create a pod which uses `emptyDir`:
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
EOF
```


https://kubernetes.io/docs/concepts/storage/volumes/#hostpath
Create a pod which uses `hostPath`:
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    hostPath:
      path: /data

EOF
```
