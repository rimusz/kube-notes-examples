# Init Containers basics

- Run sequentially to completion
- If restartPolicy is Never, and the container fails, it won't be retried.
- Don't have readiness probes
- Volumes and network are initialized first
- Restarting the pod will restart the init container
- Modifying init container image, restarts the pod
- activeDeadlineSeconds and livenessProbe will keep the init container from executing forever

Create this pod with init containers
```
cat <<EOF | kubectl create -f -

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
EOF
```

Create services for the init containers above
```
cat <<EOF | kubectl create -f -
kind: Service
apiVersion: v1
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
EOF
```

If we execute the command above, the pod will change it's status
```
kubectl get pod myapp-pod -w

NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          1m
myapp-pod   0/1       Init:1/2   0         3m
myapp-pod   0/1       Init:1/2   0         3m
```

Logs can be checked using the container name
```
kubectl logs -f myapp-pod init-myservice
kubectl logs -f myapp-pod init-myservice
```

```
cat <<EOF | kubectl create -f -
kind: Service
apiVersion: v1
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5432
EOF
```
