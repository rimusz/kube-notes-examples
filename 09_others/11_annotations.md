# Annotations

Create this pod with with annotations
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod-ann
  labels:
    app: myapp
  annotations:
    key1: value1
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
EOF
```
