# Logging/Monitoring

* https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/
* Manage cluster component logs
  * Master
    * /var/log/kube-apiserver.log - API Server, responsible for serving the API
    * /var/log/kube-scheduler.log - Scheduler, responsible for making scheduling decisions
    * /var/log/kube-controller-manager.log - Controller that manages replication controllers
  * Worker Nodes
    * /var/log/kubelet.log - Kubelet, responsible for running containers on the node
    * /var/log/kube-proxy.log - Kube Proxy, responsible for service load balancing

* [Manage application logs](https://kubernetes.io/docs/user-guide/kubectl/v1.6/#logs)
