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


https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims
Create PVC:
```
cat <<EOF | kubectl create -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  # existing claim name
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow

EOF
```

Use PVC in a pod:
```
cat <<EOF | kubectl create -f -
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: dockerfile/nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim

EOF
```
