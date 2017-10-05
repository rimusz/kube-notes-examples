# Tips

* [kubectl explain](https://blog.heptio.com/kubectl-explain-heptioprotip-ee883992a243)
* [kubectl cheatsheet](https://kubernetes.io/docs/user-guide/kubectl-cheatsheet/)
* When using kubectl for investigations and troubleshooting utilize the wide output it gives your more details
```
$ kubectl get pods -o wide --show-labels --all-namespaces
```
* In `kubectl` utilizie `--all-namespaces` to ensure deployments, pods, objects are on the right name space, and right desired state

* for events and troubleshooting utilize kubectl describe
```
$ kubectl describe pods <PODID>
```
* the '-o yaml' in conjuction with `--dry-run` allows you to create a manifest template from an imperative spec, combined with `--edit` it allows you to modify the object before creation
```
$ kubectl create service clusterip my-svc -o yaml --dry-run > /tmp/srv.yaml
$ kubectl create --edit -f /tmp/srv.yaml
```

if [ -z path]; then blah;fi

command: [“sh”, “-c”, “one line script”]

