# Create pod

1. Create yaml file
```yml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual
spec:
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
```

2. create pod
```kubectl create -f kubia-manual.yaml```

3. check pod info (build file, log)
```
kubectl get pods
k describe pod kubia-manual

kubectl get po kubia-manual -o yaml
kubectl get po kubia-manual -o json

kubectl logs kubia-manual
```

## Expose this port
```
kubectl port-forward kubia-manual 8888:8080
... Forwarding from 127.0.0.1:8888 -> 8080
... Forwarding from [::1]:8888 -> 8080
```

# Label Pods
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

## Change Labels
```shell
kubectl label po kubia-manual creation_method=manual

//You need to use the --overwrite option when changing existing labels.
kubectl label po kubia-manual-v2 env=debug --overwrite

kubectl get po -L creation_method,env
kubia-l2rmb       1/1     Running   0          56m
kubia-manual      1/1     Running   0          33m     manual
kubia-manual-v2   1/1     Running   0          2m11s   manual            debug
```

## Schedule pod to specific node
```yml
kubectl label node gke-kubia-85f6-node-0rrx gpu=true

kubectl get nodes -l gpu=true

apiVersion: v1
kind: Pod
metadata:
  name: kubia-gpu
spec:
  nodeSelector:
    gpu: "true"
  containers:
  - image: luksa/kubia
name: kubia
```

# Annotation
```
kubectl annotate pod kubia-manual mycompany.com/someannotation="foo bar"

//可以通过yaml或者describe看
k get pod kubia-manual -o yaml
kubectl describe pod kubia-manual
```

# NameSpace
```
kubectl get ns
kubectl get po --namespace kube-system
```

1. create namespace
```
kubectl create namespace custom-namespace
```
or
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: custom-namespace
```

2. creat pod
```
kubectl create -f kubia-manual.yaml -n custom-namespace
```
