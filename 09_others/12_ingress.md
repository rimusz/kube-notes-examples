# Ingress

https://kubernetes.io/docs/concepts/services-networking/ingress/

Create this nginx pod:
```
cat <<EOF | kubectl create -f -

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx

EOF
```

Create services for the pod above:
```
cat <<EOF | kubectl create -f -
kind: Service
apiVersion: v1
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

EOF
```

Create ingress for the service above:
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
EOF
```
