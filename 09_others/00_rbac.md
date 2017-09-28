## Add users to roles

List roles (namespaced)
```
kubectl get roles
```

List cluster roles
```
 kubectl get clusterroles
 ```

Assign
```
kubectl create clusterrolebinding node-proxy-bind --clusterrole=system:node-proxier --user=proxy
```
