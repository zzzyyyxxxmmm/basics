# Run nodejs on k8s

## [1. create nodejs image and push it to docker hub](https://github.com/zzzyyyxxxmmm/basics/blob/master/microservice/docker/step-by-step%3A%20nodejs.md)

## 2. create pods
```
kubectl run kubia --image=luksa/kubia --port=8080 --generator=run-pod/v1
```

It's replicationcontroller who is responsible for creating the real image. And now, pod is visibile internally.

check replicationcontrollers
```
kubectl get replicationcontrollers
```

## 3. Expose pods as a service
```
kubectl expose rc kubia --type=LoadBalancer --name kubia-http
```

## 4. Check
```
k get svc
```

## 5. Scale
```
kubectl scale rc kubia --replicas=3
```
