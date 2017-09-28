# Jobs

Create this pod run to completion and parallelism
```
cat <<EOF | kubectl create -f -
apiVersion: batch/v1
kind: Job
metadata:
  name: completion
spec:
  completions: 12
  parallelism: 3
  template:
    metadata:
      name: completion
    spec:
      containers:
      - name: completion
        image: alpine
        command: ["bin/ash",  "-c", "for i in 9 8 7 6 5; do echo $i ; done"]
      restartPolicy: Never
EOF
```


Check for jobs:
```
kubectl get jobs
```

check for pods:
```
kubectl get pods

```
