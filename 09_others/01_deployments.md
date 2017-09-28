# Basic notes on deployments

## Create deployment

Choose Strategy
- Recreate
When a release can't co-exists with the previous version

- RollingUpdate
When a release can be deployed side by side with previous version
  - max unavailable
  - max surge (how many can be created beyond the total replicas)


```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    k8s-app: nginx
  name: nginx-deployment
  namespace: default
spec:
  selector:
    matchLabels:
      k8s-app: nginx
  replicas: 2
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        k8s-app: nginx
    spec:
      containers:
      - image: nginx:1.8.0
        name: nginx
        imagePullPolicy: Always
      restartPolicy: Always
EOF
```

Update Deployment
(record will save the command at the rollout history)
```
kubectl set image deployment nginx-deployment nginx=nginx:1.9.0 --record
```

Check rollout status
```
kubectl rollout status deployment nginx-deployment

Waiting for rollout to finish: 1 old replicas are pending termination...
Waiting for rollout to finish: 1 of 2 updated replicas are available...
Waiting for rollout to finish: 1 of 2 updated replicas are available...
Waiting for rollout to finish: 1 of 2 updated replicas are available...
deployment "nginx-deployment" successfully rolled out
```

Patch deployment
```
kubectl set image deployment nginx-deployment nginx=nginx:1.9.1 --record
```

Get deployment history
```
kubectl rollout history deployment nginx-deployment

1 ...
2 ...
3 ...
```

Get deployment history detail
```
kubectl rollout history deployment nginx-deployment --revision 2

```

Rollback to previous versions
```
kubectl rollout undo deployment nginx-deployment
```

Rollback to a revision
```
kubectl rollout undo deployment nginx-deployment --to-revision 2
```
Scale
```
kubectl scale deployment nginx-deployment --replicas=10
```
