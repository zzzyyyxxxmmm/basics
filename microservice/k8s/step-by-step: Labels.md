# 1. Craete yaml file

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: kubia-manual-v2
    labels:
        creation_method: manual
        env: prod
spec:
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
```

```
kubectl get po --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
kubia-l2rmb       1/1     Running   0          54m   run=kubia
kubia-manual      1/1     Running   0          31m   <none>
kubia-manual-v2   1/1     Running   0          6s    creation_method=manual,env=prod


kubectl get po -L creation_method,env
kubia-l2rmb       1/1     Running   0          55m
kubia-manual      1/1     Running   0          31m
kubia-manual-v2   1/1     Running   0          30s   manual            prod
```

# 2. Change Labels
```shell
kubectl label po kubia-manual creation_method=manual

//You need to use the --overwrite option when changing existing labels.
kubectl label po kubia-manual-v2 env=debug --overwrite

kubectl get po -L creation_method,env
kubia-l2rmb       1/1     Running   0          56m
kubia-manual      1/1     Running   0          33m     manual
kubia-manual-v2   1/1     Running   0          2m11s   manual            debug
```