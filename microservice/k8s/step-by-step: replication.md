1. create yaml file
```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    app: kubia
template:
  metadata:
    labels:
      app: kubia
  spec:
    containers:
    - name: kubia
      image: luksa/kubia`
      ports:
      - containerPort: 8080
```

2. check pods

3. change pod labels so they will be out of the scope of a ReplicationController

4. delete
```
kubectl delete rc kubia --cascade=false
```

# ReplicaSet
```yaml
selector:
  matchExpressions:
    - key: app
      operator: In
      values:
- kubia
```

# deamonSet
```yml
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk: ssd
      containers:
- name: main
        image: luksa/ssd-monitor
```
You’re defining a DaemonSet that will run a pod with a single container based on the luksa/ssd-monitor container image. An instance of this pod will be created for each node that has the disk=ssd label.

Now, imagine you’ve made a mistake and have mislabeled one of the nodes. It has a spinning disk drive, not an SSD. What happens if you change the node’s label? The pod is being terminated. But you knew that was going to happen, right? This wraps up your exploration of DaemonSets, so you may want to delete your ssd-monitor DaemonSet. If you still have any other daemon pods running, you’ll see that deleting the DaemonSet deletes those pods as well.