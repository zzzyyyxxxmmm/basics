# Creating an HTTP-based liveness probe

### create yaml
```yml
apiVersion: v1
kind: pod
metadata:
  name: kubia-liveness
spec:
  containers:
  - image: luksa/kubia-unhealthy
    name: kubia
    livenessProbe:
      httpGet:
        path: /
      initialDelaySeconds: 15
port: 8080
```


