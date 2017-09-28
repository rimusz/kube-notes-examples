# Basic notes on pdb

Limits the number of pods that can be removed simultaneously

kubectl drain (uses eviction API)
kubectl delete (does not use eviction API)

Pod unavailable or deleted count against the disruption budget

You can set-cluster `minAvailabe` or `maxUnavailable`, not both
(maxUnavailable requires > 1.7)


```
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: zookeeper
```

Check status
```
kubectl get poddisruptionbudgets
```
