# Overview
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_arch.png" width="700" height="500">
</div>

### Master
* **The Kubernetes API Server**, which you and the other Control Plane components communicate with.
* **The Scheduler**, which schedules your apps (assigns a worker node to each deploy- able component of your application)
* **The Controller Manager**, which performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on
* **etcd**, a reliable distributed data store that persistently stores the cluster configuration.

### Worker
* Docker, rkt, or another **container runtime**, which runs your containers
* **The Kubelet**, which talks to the API server and manages containers on its node
* **The Kubernetes Service Proxy** (kube-proxy), which load-balances network traffic between application components

## Running an application in Kubernetes

1. **create docker image and push it to registry**

To run an application in Kubernetes, you first need to package it up into one or more container images, push those images to an image registry.

2. **Post a description of your app to the Kubernetes API server.**

The description includes information such as the container image or images that contain your application components, how those components are related to each other, and which ones need to be run co-located (together on the same node) and which don’t. For each component, you can also specify how many copies (or replicas) you want to run. Additionally, the description also includes which of those compo- nents provide a service to either internal or external clients and should be exposed through a single IP address and made discoverable to the other components.

3. **Scheduler schedules according to the description**

When the API server processes your app’s description, the Scheduler schedules the specified groups of containers onto the available worker nodes based on computa- tional resources required by each group and the unallocated resources on each node

4. **Kubelet pull and run**

The Kubelet on those nodes then instructs the Container Runtime (Docker, for example) to pull the required container images and run the containers.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_example.png" width="700" height="500">
</div>

The app descriptor lists four containers, grouped into three sets (these sets are called pods; we’ll explain what they are in chapter 3). The first two pods each contain only a single container, whereas the last one contains two. That means both containers need to run co-located and shouldn’t be isolated from each other. Next to each pod, you also see a number representing the number of replicas of each pod that need to run in parallel. After submitting the descriptor to Kuberne- tes, it will schedule the specified number of replicas of each pod to the available worker nodes. The Kubelets on the nodes will then tell Docker to pull the container images from the image registry and run the containers.

### Run one image
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_run_image.png" width="700" height="500">
</div>
First, you built the image and pushed it to Docker Hub. This was necessary because building the image on your local machine only makes it available on your local machine, but you needed to make it accessible to the Docker daemons running on your worker nodes.
When you ran the kubectl command, it created a new ReplicationController object in the cluster by sending a REST HTTP request to the Kubernetes API server. The ReplicationController then created a new pod, which was then scheduled to one of the worker nodes by the Scheduler. The Kubelet on that node saw that the pod was scheduled to it and instructed Docker to pull the specified image from the registry because the image wasn’t available locally. After downloading the image, Docker cre- ated and ran the container.

# PODS
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_pod.png" width="500" height="300">
</div>

A pod is a group of one or more tightly related containers that will always run together on the same worker node and in the same Linux namespace(s). Each pod is like a separate logical machine with its own IP, hostname, processes, and so on, running a single application. The application can be a single process, running in a single container, or it can be a main application process and additional supporting processes, each running in its own container. All the containers in a pod will appear to be running on the same logical machine, whereas containers in other pods, even if they’re running on the same worker node, will appear to be running on a different one. 

The key thing about pods is that when a pod does contain multiple con- tainers, all of them are always run on a single worker node—it never spans multiple worker nodes.

Because you’re not supposed to group multiple processes into a single container, it’s obvious you need another higher-level construct that will allow you to bind containers together and manage them as a single unit. This is the reasoning behind pods.
A pod of containers allows you to run closely related processes together and provide them with (almost) the same environment as if they were all running in a single container, while keeping them somewhat isolated. This way, you get the best of both worlds. You can take advantage of all the features containers provide, while at the same time giving the processes the illusion of running together.

Because all containers of a pod run under the same Network and UTS namespaces (we’re talking about Linux namespaces here), they all share the same hostname and network interfaces. Similarly, all containers of a pod run under the same IPC namespace and can communicate through IPC. In the latest Kubernetes and Docker versions, they can also share the same PID namespace, but that feature isn’t enabled by default.

One thing to stress here is that because containers in a pod run in the same Network namespace, they share the same IP address and port space. This means processes running in containers of the same pod need to take care not to bind to the same port numbers or they’ll run into port conflicts. But this only concerns containers in the same pod. Containers of different pods can never run into port conflicts, because each pod has a separate port space. All the containers in a pod also have the same loopback network interface, so a container can communicate with other containers in the same pod through localhost.

Pod runs like a single machine, the containers in a pod run co-located.

### (POD)Do you think a multi-tier application consisting of a frontend application server and a backend database should be configured as a single pod or as two pods?
**Although nothing** is stopping you from running both the frontend server and the database in a single pod with two containers, it isn’t the most appropriate way. We’ve said that all containers of the same pod always run co-located, but do the web server and the database really need to run on the same machine? The answer is obviously no, so you don’t want to put them into a single pod. But is it wrong to do so regardless? In a way, it is.

If both the frontend and backend are in the same pod, then both will always be run on the same machine. If you **have a two-node Kubernetes cluster and only this single pod**, you’ll only be using a single worker node and not taking advantage of the computational resources (CPU and memory) you have at your disposal on the second node. Splitting the pod into two would allow Kubernetes to schedule the frontend to one node and the backend to the other node, thereby improving the utilization of your infrastructure.

**Another reason** why you shouldn’t put them both into a single pod is scaling. A pod is also the basic unit of scaling. Kubernetes can’t horizontally scale individual contain- ers; instead, it scales whole pods. If your pod consists of a frontend and a backend con- tainer, when you scale up the number of instances of the pod to, let’s say, two, you end up with two frontend containers and two backend containers.
Usually, frontend components have completely different scaling requirements than the backends, so we tend to scale them individually. Not to mention the fact that backends such as databases are usually much harder to scale compared to (stateless) frontend web servers. If you need to scale a container individually, this is a clear indi- cation that it needs to be deployed in a separate pod.

To recap how containers should be grouped into pods—when deciding whether to put two containers into a single pod or into two separate pods, you always need to ask yourself the following questions:
* Do they need to be run together or can they run on different hosts?
* Do they represent a single whole or are they independent components? 
* Must they be scaled together or individually?

## Pod definition
```yaml
k get po kubia-52k5b -o yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2019-11-03T21:19:20Z"
  generateName: kubia-
  labels:
    run: kubia
  name: kubia-52k5b
  namespace: default
  ownerReferences:
  - apiVersion: v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicationController
    name: kubia
    uid: 8979d484-ea2f-492f-bd22-2ed6da6b6a42
  resourceVersion: "83297"
  selfLink: /api/v1/namespaces/default/pods/kubia-52k5b
  uid: f8cdc22d-41a4-470b-8af5-bdf685fd7c78
spec:
  containers:
  - image: zzzyyyxxxmmm/kubia
    imagePullPolicy: Always
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-jm77n
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-jm77n
    secret:
      defaultMode: 420
      secretName: default-token-jm77n
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2019-11-03T21:19:20Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2019-11-03T21:19:22Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2019-11-03T21:19:22Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2019-11-03T21:19:20Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://8ada8e065cc1f6afa834ab344cbf3b113751b93583d6cb51621f7b1bd7062dd3
    image: kubia:latest
    imageID: docker-pullable://zzzyyyxxxmmm/kubia@sha256:c637cdf07feeab097886f33e36c4c156a2a182f84a12191f933a404f4740ced8
    lastState: {}
    name: kubia
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2019-11-03T21:19:22Z"
  hostIP: 192.168.31.132
  phase: Running
  podIP: 172.17.0.6
  podIPs:
  - ip: 172.17.0.6
  qosClass: BestEffort
  startTime: "2019-11-03T21:19:20Z"
```

The pod definition consists of a few parts. First, there’s the Kubernetes API version used in the YAML and the type of resource the YAML is describing. Then, three important sections are found in almost all Kubernetes resources:
* Metadata includes the name, namespace, labels, and other information about the pod.
* Spec contains the actual description of the pod’s contents, such as the pod’s con- tainers, volumes, and other data.
* Status contains the current information about the running pod, such as what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info.

### custom pod yaml
```yaml
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

```
kubectl create -f kubia-manual.yaml
kubectl get po kubia-manual -o yaml
kubectl get po kubia-manual -o json
kubectl logs kubia-manual
```

## 直接和pods通信
```
kubectl port-forward kubia-manual 8888:8080
curl localhost:8888
```

## Labels

For example, with microservices architectures, the number of deployed microser- vices can easily exceed 20 or more. Those components will probably be replicated (multiple copies of the same component will be deployed) and multiple versions or releases (stable, beta, canary, and so on) will run concurrently. This can lead to hun- dreds of pods in the system. Without a mechanism for organizing them, you end up with a big, incomprehensible mess, such as the one shown in figure 3.6. The figure shows pods of multiple microservices, with several running multiple replicas, and others running different releases of the same microservice.

Labels are a simple, yet incredibly powerful, Kubernetes feature for organizing not only pods, but all other Kubernetes resources. A label is an arbitrary key-value pair you attach to a resource, which is then utilized when selecting resources using label selectors (resources are filtered based on whether they include the label specified in the selec- tor). A resource can have more than one label, as long as the keys of those labels are unique within that resource. You usually attach labels to resources when you create them, but you can also add additional labels or even modify the values of existing labels later without having to recreate the resource.

[With and Without Labels](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_arch.png)


# SERVICES
The expose command’s output mentions a service called kubia-http. Services are objects like Pods and Nodes, so you can see the newly created Service object by run- ning the kubectl get services command

# Kubectl

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_kubectl.png" width="700" height="500">
</div>
