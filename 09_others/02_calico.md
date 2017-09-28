## Install Calico

Create configmap to configure calico (with kubernetes storage)
```
cat <<EOF | kubectl create -f -
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-calico-cfg
  namespace: kube-system
data:
  # The CNI network configuration to install on each node.
  cni_network_config: |-
    {
        "name": "k8s-pod-network",
        "cniVersion": "0.3.0",
        "type": "calico",
        "log_level": "debug",
        "datastore_type": "kubernetes",
        "nodename": "__KUBERNETES_NODE_NAME__",
        "ipam": {
            "type": "host-local",
            "subnet": "usePodCidr"
        },
        "policy": {
            "type": "k8s",
            "k8s_auth_token": "__SERVICEACCOUNT_TOKEN__"
        },
        "kubernetes": {
            "k8s_api_root": "https://__KUBERNETES_SERVICE_HOST__:__KUBERNETES_SERVICE_PORT__",          
            "kubeconfig": "__KUBECONFIG_FILEPATH__"
        }
    }
EOF
```

Create service account
```
cat <<EOF | kubectl create -f -
kind: ServiceAccount
metadata:
 name: kube-calico
 namespace: kube-system
EOF
```

Create role
```
cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: kube-calico
  namespace: kube-system
rules:
  - apiGroups: [""]
    resources:
      - namespaces
    verbs:
      - get
      - list
      - watch
  - apiGroups: [""]
    resources:
      - pods/status
    verbs:
      - update
  - apiGroups: [""]
    resources:
      - pods
    verbs:
      - get
      - list
      - watch
  - apiGroups: [""]
    resources:
      - nodes
    verbs:
      - get
      - list
      - update
      - watch
  - apiGroups: ["extensions"]
    resources:
      - thirdpartyresources
    verbs:
      - create
      - get
      - list
      - watch
  - apiGroups: ["extensions"]
    resources:
      - networkpolicies
    verbs:
      - get
      - list
      - watch
  - apiGroups: ["projectcalico.org"]
    resources:
      - globalconfigs
    verbs:
      - create
      - get
      - list
      - update
      - watch
  - apiGroups: ["projectcalico.org"]
    resources:
      - ippools
    verbs:
      - create
      - delete
      - get
      - list
      - update
      - watch
  - apiGroups: ["alpha.projectcalico.org"]
    resources:
      - systemnetworkpolicies
    verbs:
      - get
      - list
      - watch
EOF
```

Bind role to service account
```
cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kube-calico
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-calico
subjects:
- kind: ServiceAccount
  name: kube-calico
  namespace: kube-system
EOF
```

Deploy Calico daemonset
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: kube-calico
  namespace: kube-system
  labels:
    k8s-app: kube-calico
spec:
  selector:
    matchLabels:
      k8s-app: kube-calico
  template:
    metadata:
      labels:
        k8s-app: kube-calico
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      hostNetwork: true
      serviceAccountName: kube-calico
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
        - key: "CriticalAddonsOnly"
          operator: "Exists"
      containers:
        - name: kube-calico
          image: quay.io/calico/node:v1.3.0
          env:
            - name: DATASTORE_TYPE
              value: "kubernetes"
            - name: FELIX_LOGSEVERITYSCREEN
              value: "info"
            - name: CALICO_NETWORKING_BACKEND
              value: "none"
            - name: CALICO_DISABLE_FILE_LOGGING
              value: "true"
            - name: FELIX_DEFAULTENDPOINTTOHOSTACTION
              value: "ACCEPT"
            - name: FELIX_IPV6SUPPORT
              value: "false"
            - name: WAIT_FOR_DATASTORE
              value: "true"
            - name: CALICO_IPV4POOL_CIDR
              value: "10.2.0.0/16"
            - name: CALICO_IPV4POOL_IPIP
              value: "always"
            - name: NODENAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: IP
              value: ""
          securityContext:
            privileged: true
          resources:
            requests:
              cpu: 250m
          volumeMounts:
            - mountPath: /var/run/calico
              name: var-run-calico
              readOnly: false
        - name: install-cni
          image: quay.io/calico/cni:v1.9.1-4-g23fcd5f
          command: ["/install-cni.sh"]
          env:
            - name: CNI_NETWORK_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: kube-calico-cfg
                  key: cni_network_config
            - name: CNI_NET_DIR
              value: "/etc/kubernetes/cni/net.d"
            - name: KUBERNETES_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: SKIP_CNI_BINARIES
              value: bridge,cnitool,dhcp,flannel,host-local,ipvlan,loopback,macvlan,noop,portmap,ptp,tuning
          volumeMounts:
            - mountPath: /host/opt/cni/bin
              name: cni-bin-dir
            - mountPath: /host/etc/cni/net.d
              name: cni-net-dir
      volumes:
        - name: var-run-calico
          hostPath:
            path: /var/run/calico
        - name: cni-bin-dir
          hostPath:
            path: /opt/cni/bin
        - name: cni-net-dir
          hostPath:
            path: /etc/kubernetes/cni/net.d
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
EOF
```

Calico policy controller
(at master! check the etcd location, nodeselector, volume mounts ...)
```
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: calico-policy-controller
  namespace: kube-system
  labels:
    k8s-app: calico-policy
  annotations:
    scheduler.alpha.kubernetes.io/critical-pod: ''
    scheduler.alpha.kubernetes.io/tolerations: |
      [{"key": "dedicated", "value": "master", "effect": "NoSchedule" },
       {"key":"CriticalAddonsOnly", "operator":"Exists"}]
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      name: calico-policy-controller
      namespace: kube-system
      labels:
        k8s-app: calico-policy
    spec:
      hostNetwork: true
      nodeSelector:
        kubernetes.io/hostname: kube-01
      containers:
        - name: calico-policy-controller
          image: calico/kube-policy-controller:v0.5.2
          env:
            # The location of the Calico etcd cluster.
            - name: ETCD_ENDPOINTS
              value: "https://10.132.109.203:2379"
            - name: ETCD_CA_CERT_FILE
              value: /calico-secrets/ca.pem
            - name: ETCD_KEY_FILE
              value: /calico-secrets/apiserver-key.pem
            # Location of the client certificate for etcd.
            - name: ETCD_CERT_FILE
              value: /calico-secrets/apiserver.pem
            - name: K8S_API
              value: "https://kubernetes.default:443"
            - name: CONFIGURE_ETC_HOSTS
              value: "true"
          volumeMounts:
            # Mount in the etcd TLS secrets.
            - mountPath: /calico-secrets
              name: etcd-certs
      volumes:
        # Mount in the etcd TLS secrets.
        - name: etcd-certs
          hostPath:
            path: /var/lib/kubernetes
EOF
```
