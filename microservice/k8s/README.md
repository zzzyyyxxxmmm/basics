# Overview
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_arch.png" width="700" height="500">
</div>

### Master
* **The Kubernetes API Server**, which you and the other Control Plane components communicate with.
* **The Scheduler**, which schedules your apps (assigns a worker node to each deployable component of your application)
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

The description includes information such as the container image or images that contain your application components, how those components are related to each other, and which ones need to be run co-located (together on the same node) and which don’t. For each component, you can also specify how many copies (or replicas) you want to run. Additionally, the description also includes which of those components provide a service to either internal or external clients and should be exposed through a single IP address and made discoverable to the other components.

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

The key thing about pods is that when a pod does contain multiple containers, all of them are always run on a single worker node—it never spans multiple worker nodes.

Because you’re not supposed to group multiple processes into a single container, it’s obvious you need another higher-level construct that will allow you to bind containers together and manage them as a single unit. This is the reasoning behind pods.

A pod of containers allows you to run closely related processes together and provide them with (almost) the same environment as if they were all running in a single container, while keeping them somewhat isolated. This way, you get the best of both worlds. You can take advantage of all the features containers provide, while at the same time giving the processes the illusion of running together. 

Because all containers of a pod run under the same Network, UTS namespaces cgroups, they all share the same hostname and network interfaces. Similarly, all containers of a pod run under the same IPC namespace and can communicate through IPC. In the latest Kubernetes and Docker versions, they can also share the same PID namespace, but that feature isn’t enabled by default. Containers in different Pods have distinct IP addresses and can not communicate by IPC without special configuration. These containers usually communicate with each other via Pod IP addresses.

One thing to stress here is that because containers in a pod run in the same Network namespace, they share the same IP address and port space (talk by localhost, standard inter-process communications like SystemV semaphores or POSIX shared memory). This means processes running in containers of the same pod need to take care not to bind to the same port numbers or they’ll run into port conflicts. But this only concerns containers in the same pod. Containers of different pods can never run into port conflicts, because each pod has a separate port space. All the containers in a pod also have the same loopback network interface, so a container can communicate with other containers in the same pod through localhost.

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

## Termination of a pod
Because Pods represent running processes on nodes in the cluster, it is important to allow those processes to gracefully terminate when they are no longer needed (vs being violently killed with a KILL signal and having no chance to clean up). Users should be able to request deletion and know when processes terminate, but also be able to ensure that deletes eventually complete. **When a user requests deletion of a Pod, the system records the intended grace period before the Pod is allowed to be forcefully killed, and a TERM signal is sent to the main process in each container. Once the grace period has expired, the KILL signal is sent to those processes, and the Pod is then deleted from the API server.** If the Kubelet or the container manager is restarted while waiting for processes to terminate, the termination will be retried with the full grace period.

An example flow:

1. User sends command to delete Pod, with default grace period (30s)
2. The Pod in the API server is updated with the time beyond which the Pod is considered “dead” along with the grace period.
3. Pod shows up as “Terminating” when listed in client commands
4. (simultaneous with 3) When the Kubelet sees that a Pod has been marked as terminating because the time in 2 has been set, it begins the Pod shutdown process.
  * If one of the Pod’s containers has defined a preStop hook, it is invoked inside of the container. If the preStop hook is still running after the grace period expires, step 2 is then invoked with a small (2 second) extended grace period.
  * The container is sent the TERM signal. Note that not all containers in the Pod will receive the TERM signal at the same time and may each require a preStop hook if the order in which they shut down matters.
5. (simultaneous with 3) Pod is removed from endpoints list for service, and are no longer considered part of the set of running Pods for replication controllers. Pods that shutdown slowly cannot continue to serve traffic as load balancers (like the service proxy) remove them from their rotations.
6. When the grace period expires, any processes still running in the Pod are killed with SIGKILL.
7. The Kubelet will finish deleting the Pod on the API server by setting grace period 0 (immediate deletion). The Pod disappears from the API and is no longer visible from the client.
  
By default, all deletes are graceful within 30 seconds. The kubectl delete command supports the --grace-period=<seconds> option which allows a user to override the default and specify their own value. The value 0 force deletes the Pod. You must specify an additional flag --force along with --grace-period=0 in order to perform force deletions.

### Force deletion of pods
Force deletion of a Pod is defined as deletion of a Pod from the cluster state and etcd immediately. When a force deletion is performed, the API server does not wait for confirmation from the kubelet that the Pod has been terminated on the node it was running on. It removes the Pod in the API immediately so a new Pod can be created with the same name. On the node, Pods that are set to terminate immediately will still be given a small grace period before being force killed.

Force deletions can be potentially dangerous for some Pods and should be performed with caution. In case of StatefulSet Pods, please refer to the task documentation for deleting Pods from a StatefulSet.



## Labels

For example, with microservices architectures, the number of deployed microser- vices can easily exceed 20 or more. Those components will probably be replicated (multiple copies of the same component will be deployed) and multiple versions or releases (stable, beta, canary, and so on) will run concurrently. This can lead to hun- dreds of pods in the system. Without a mechanism for organizing them, you end up with a big, incomprehensible mess, such as the one shown in figure 3.6. The figure shows pods of multiple microservices, with several running multiple replicas, and others running different releases of the same microservice.

Labels are a simple, yet incredibly powerful, Kubernetes feature for organizing not only pods, but all other Kubernetes resources. A label is an arbitrary key-value pair you attach to a resource, which is then utilized when selecting resources using label selectors (resources are filtered based on whether they include the label specified in the selec- tor). A resource can have more than one label, as long as the keys of those labels are unique within that resource. You usually attach labels to resources when you create them, but you can also add additional labels or even modify the values of existing labels later without having to recreate the resource.

[With and Without Labels](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_arch.png)

You never want to say specifically what node a pod should be scheduled to, because that would couple the application to the infrastructure, whereas the whole idea of Kubernetes is hiding the actual infrastructure from the apps that run on it. But if you want to have a say in where a pod should be scheduled, instead of specifying an exact node, you should describe the node requirements and then let Kubernetes select a node that matches those requirements. This can be done through node labels and node label selectors.

### Annotating pods
In addition to labels, pods and other objects can also contain annotations. Annotations are also key-value pairs, so in essence, they’re similar to labels, but they aren’t meant to hold identifying information. They can’t be used to group objects the way labels can. While objects can be selected through label selectors, there’s no such thing as an annotation selector.
On the other hand, annotations can hold much larger pieces of information and are primarily meant to be used by tools. Certain annotations are automatically added to objects by Kubernetes, but others are added by users manually.

### Using namespaces to group resources
Let’s turn back to labels for a moment. We’ve seen how they organize pods and other objects into groups. Because each object can have multiple labels, those groups of objects can overlap. Plus, when working with the cluster (through kubectl for example), if you don’t explicitly specify a label selector, you’ll always see all objects. But what about times when you want to split objects into separate, non-overlapping groups? You may want to only operate inside one group at a time. For this and other reasons, Kubernetes also groups objects into namespaces.

这时候就可以利用namespace建立prod, test, dev环境

Using multiple namespaces allows you to split complex systems with numerous components into smaller distinct groups. They can also be used for separating resources in a multi-tenant environment, splitting up resources into production, development, and QA environments, or in any other way you may need. Resource names only need to be unique within a namespace. Two different namespaces can contain resources of the same name. But, while most types of resources are namespaced, a few aren’t. One of them is the Node resource, which is global and not tied to a single namespace. (persistentVolumes也是)

并不是真正意义上网络的隔离: To wrap up this section about namespaces, let me explain what namespaces don’t provide—at least not out of the box. Although namespaces allow you to isolate objects into distinct groups, which allows you to operate only on those belonging to the specified namespace, they don’t provide any kind of isolation of running objects.
For example, you may think that when different users deploy pods across different namespaces, those pods are isolated from each other and can’t communicate, but that’s not necessarily the case. Whether namespaces provide network isolation depends on which networking solution is deployed with Kubernetes. When the solution doesn’t provide inter-namespace network isolation, if a pod in namespace foo knows the IP address of a pod in namespace bar, there is nothing preventing it from sending traffic, such as HTTP requests, to the other pod.

(namespaces-walkthrough)[https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/]

# Replication and other controllers: deploying managed pods

## Keeping pods healthy
If pod crash, k8s will restart it automatically. What if they died sliently because of a infinit loop? Kubernetes can check if a container is still alive through liveness probes. You can specify a liveness probe for each container in the pod’s specification. 
* An HTTP GET probe performs an HTTP GET request on the container’s IP address, a port and path you specify. If the probe receives a response, and the response code doesn’t represent an error (in other words, if the HTTP response code is 2xx or 3xx), the probe is considered successful. If the server returns an error response code or if it doesn’t respond at all, the probe is considered a fail- ure and the container will be restarted as a result.
* A TCP Socket probe tries to open a TCP connection to the specified port of the container. If the connection is established successfully, the probe is successful. Otherwise, the container is restarted.
* An Exec probe executes an arbitrary command inside the container and checks the command’s exit status code. If the status code is 0, the probe is successful. All other codes are considered failures.

But for a better liveness check, you’d configure the probe to perform requests on a specific URL path (/health, for example) and have the app perform an internal sta- tus check of all the vital components running inside the app to ensure none of them has died or is unresponsive.

You now understand that Kubernetes keeps your containers running by restarting them if they crash or if their liveness probes fail. This job is performed by the Kubelet on the node hosting the pod—the Kubernetes Control Plane components running on the master(s) have no part in this process.

But if the node itself crashes, it’s the Control Plane that must create replacements for all the pods that went down with the node. It doesn’t do that for pods that you create directly. Those pods aren’t managed by anything except by the Kubelet, but because the Kubelet runs on the node itself, it can’t do anything if the node fails.
To make sure your app is restarted on another node, you need to have the pod managed by a ReplicationController or similar mechanism, which we’ll discuss in the rest of this chapter.

Be sure to check only the internals of the app and nothing influenced by an external factor. For example, a frontend web server’s liveness probe shouldn’t return a failure when the server can’t connect to the backend database. If the underlying cause is in the database itself, restarting the web server container will not fix the problem. Because the liveness probe will fail again, you’ll end up with the container restarting repeatedly until the database becomes accessible again.

## Introducing ReplicationControllers
A ReplicationController is a Kubernetes resource that ensures its pods are always kept running. If the pod disappears for any reason, such as in the event of a node disappearing from the cluster or because the pod was evicted from the node, the ReplicationController notices the missing pod and creates a replacement pod.

Like many things in Kubernetes, a ReplicationController, although an incredibly sim- ple concept, provides or enables the following powerful features:
* It makes sure a pod (or multiple pod replicas) is always running by starting a new pod when an existing one goes missing.
* When a cluster node fails, it creates replacement replicas for all the pods that were running on the failed node (those that were under the Replication- Controller’s control).
* It enables easy horizontal scaling of pods—both manual and automatic.

A ReplicationController’s job is to make sure that an exact number of pods always matches its label selector. A ReplicationController has three essential parts:
* A label selector, which determines what pods are in the ReplicationController’s scope 
* A replica count, which specifies the desired number of pods that should be running 
* A pod template, which is used when creating new pod replicas

## Using ReplicaSets instead of ReplicationControllers
A ReplicaSet behaves exactly like a ReplicationController, but it has more expressive pod selectors. Whereas a ReplicationController’s label selector only allows matching pods that include a certain label, a ReplicaSet’s selector also allows matching pods that lack a certain label or pods that include a certain label key, regardless of its value.

Also, for example, a single ReplicationController can’t match pods with the label env=production and those with the label env=dev at the same time. It can only match either pods with the env=production label or pods with the env=devel label. But a single ReplicaSet can match both sets of pods and treat them as a single group.

Similarly, a ReplicationController can’t match pods based merely on the presence of a label key, regardless of its value, whereas a ReplicaSet can. For example, a Replica- Set can match all pods that include a label with the key env, whatever its actual value is (you can think of it as env=*).

```
selector:
  matchExpressions:
    - key: app
      operator: In
      values:
        - kubia
```
You can add additional expressions to the selector. As in the example, each expression must contain a key, an operator, and possibly (depending on the operator) a list of values. You’ll see four valid operators:
* In—Label’s value must match one of the specified values.
* NotIn—Label’s value must not match any of the specified values.
* Exists—Pod must include a label with the specified key (the value isn’t importannt. When using this operator, you shouldn’t specify the values field.
* DoesNotExist—Pod must not include a label with the specified key. The values
property must not be specified.

## Running exactly one pod on each node with DaemonSets
To run a pod on all cluster nodes, you create a DaemonSet object, which is much like a ReplicationController or a ReplicaSet, except that pods created by a DaemonSet already have a target node specified and skip the Kubernetes Scheduler. They aren’t scattered around the cluster randomly.

A DaemonSet makes sure it creates as many pods as there are nodes and deploys each one on its own node.

Whereas a ReplicaSet (or ReplicationController) makes sure that a desired number of pod replicas exist in the cluster, a DaemonSet doesn’t have any notion of a desired replica count. It doesn’t need it because its job is to ensure that a pod matching its pod selector is running on each node. 

If a node goes down, the DaemonSet doesn’t cause the pod to be created elsewhere. But when a new node is added to the cluster, the DaemonSet immediately deploys a new pod instance to it. It also does the same if someone inadvertently deletes one of the pods, leaving the node without the DaemonSet’s pod. Like a ReplicaSet, a DaemonSet creates the pod from the pod template configured in it.

## Running pods that perform a single completable task
Jobs

## Scheduling Jobs to run periodically or once in the future
CronJob

## Deployments

### Updates
Here you see that when you first created the Deployment, it created a ReplicaSet (nginx-deployment-2035384211) and scaled it up to 3 replicas directly. When you updated the Deployment, it created a new ReplicaSet (nginx-deployment-1564180365) and scaled it up to 1 and then scaled down the old ReplicaSet to 2, so that at least 2 Pods were available and at most 4 Pods were created at all times. It then continued scaling up and down the new and the old ReplicaSet, with the same rolling update strategy. Finally, you’ll have 3 available replicas in the new ReplicaSet, and the old ReplicaSet is scaled down to 0.



# SERVICES
A Kubernetes Service is a resource you create to make a single, constant point of entry to a group of pods providing the same service. Each service has an IP address and port that never change while the service exists. Clients can open connections to that IP and port, and those connections are then routed to one of the pods backing that service. This way, clients of a service don’t need to know the location of individual pods providing the service, allowing those pods to be moved around the cluster at any time.
* 提供固定的 IP。由于 Pod 可以随时启停，Pod IP 可能随时都会变化，例如上面 nginx pod 重启之后 IP 可能不再是 172.17.0.11。Service 为 Pods 提供的固定 IP，其他服务可以通过 Service IP 找到提供服务的 Pods。
* 提供负载均衡。Service 由多个 Pods 组成，kubernetes 对组成 Service 的 Pods 提供的负载均衡方案，例如随机访问、基于 Client IP 的 session affinity。
* 服务发现。集群中其他服务可以通过 Service 名字访问后端服务（DNS），也可以通过环境变量访问。

## Discovering services(Pods talk to outside)
By creating a service, you now have a single and stable IP address and port that you can hit to access your pods. This address will remain unchanged throughout the whole lifetime of the service. Pods behind this service may come and go, their IPs may change, their number can go up or down, but they’ll always be accessible through the service’s single and constant IP address.
But how do the client pods know the IP and port of a service? Do you need to create the service first, then manually look up its IP address and pass the IP to the config- uration options of the client pod? Not really. Kubernetes also provides ways for client pods to discover a service’s IP and port.

### DISCOVERING SERVICES THROUGH ENVIRONMENT VARIABLES
When a pod is started, Kubernetes initializes a set of environment variables pointing to each service that exists at that moment. If you create the service before creating the client pods, processes in those pods can get the IP address and port of the service by inspecting their environment variables.
```
$ kubectl exec kubia-3inly env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubia-3inly
KUBERNETES_SERVICE_HOST=10.111.240.1 
KUBERNETES_SERVICE_PORT=443
... 
KUBIA_SERVICE_HOST=10.111.249.153 
KUBIA_SERVICE_PORT=80
```

### DISCOVERING SERVICES THROUGH DNS
Remember in chapter 3 when you listed pods in the kube-system namespace? One of the pods was called kube-dns. The kube-system namespace also includes a corresponding service with the same name.

As the name suggests, the pod runs a DNS server, which all other pods running in the cluster are automatically configured to use (Kubernetes does that by modifying each container’s /etc/resolv.conf file). Any DNS query performed by a process running in a pod will be handled by Kubernetes’ own DNS server, which knows all the services running in your system.

Each service gets a DNS entry in the internal DNS server, and client pods that know the name of the service can access it through its fully qualified domain name (FQDN) instead of resorting to environment variables.

## Connecting to services living outside the cluster

### Introducing service endpoints
Services don’t link to pods directly. Instead, a resource sits in between the Endpoints resource. You may have already noticed endpoints if you used the kubectl describe command on your service, as shown in the following listing.
```s
Name:  kubia
Namespace: default
Labels: <none>
Selector: app=kubia
Type: ClusterIP
IP: 10.111.249.153
Port: <unset> 80/TCP
Endpoints: 10.108.1.4:8080,10.108.2.5:8080,10.108.2.6:8080
Session Affinity: None
```

```
kubectl get endpoints kubia
```

## Exposing services to external clients
You have a few ways to make a service accessible externally:
* Setting the service type to NodePort—For a NodePort service, each cluster node opens a port on the node itself (hence the name) and redirects traffic received on that port to the underlying service. The service isn’t accessible only at the internal cluster IP and port, but also through a dedicated port on all nodes.
* Setting the service type to LoadBalancer, an extension of the NodePort type—This makes the service accessible through a dedicated load balancer, provisioned from the cloud infrastructure Kubernetes is running on. The load balancer redirects traffic to the node port across all the nodes. Clients connect to the service through the load balancer’s IP.
* Creating an Ingress resource, a radically different mechanism for exposing multiple ser- vices through a single IP address—It operates at the HTTP level (network layer 7) and can thus offer more features than layer 4 services can. We’ll explain Ingress resources in section 5.4.

## Signaling when a pod is ready to accept connections
There’s one more thing we need to cover regarding both Services and Ingresses. You’ve already learned that pods are included as endpoints of a service if their labels match the service’s pod selector. As soon as a new pod with proper labels is created, it becomes part of the service and requests start to be redirected to the pod. But what if the pod isn’t ready to start serving requests immediately?

The pod may need time to load either configuration or data, or it may need to perform a warm-up procedure to prevent the first user request from taking too long and affecting the user experience. In such cases you don’t want the pod to start receiving requests immediately, especially when the already-running instances can process requests properly and quickly. It makes sense to not forward requests to a pod that’s in the process of starting up until it’s fully ready.

Imagine that a group of pods (for example, pods running application servers) depends on a service provided by another pod (a backend database, for example). If at any point one of the frontend pods experiences connectivity problems and can’t reach the database anymore, it may be wise for its readiness probe to signal to Kubernetes that the pod isn’t ready to serve any requests at that time. If other pod instances aren’t experiencing the same type of connectivity issues, they can serve requests normally. A readiness probe makes sure clients only talk to those healthy pods and never notice there’s anything wrong with the system.

Unlike liveness probes, if a container fails the readiness check, it won’t be killed or restarted. This is an important distinction between liveness and readiness probes.

## Using a headless service for discovering individual pods
You’ve seen how services can be used to provide a stable IP address allowing clients to connect to pods (or other endpoints) backing each service. Each connection to the service is forwarded to one randomly selected backing pod. But what if the client needs to connect to all of those pods? What if the backing pods themselves need to each connect to all the other backing pods? Connecting through the service clearly isn’t the way to do this. What is?

For a client to connect to all pods, it needs to figure out the the IP of each individual pod. One option is to have the client call the Kubernetes API server and get the list of pods and their IP addresses through an API call, but because you should always strive to keep your apps Kubernetes-agnostic, using the API server isn’t ideal.

Luckily, Kubernetes allows clients to discover pod IPs through DNS lookups. Usually, when you perform a DNS lookup for a service, the DNS server returns a single IP—the service’s cluster IP. But if you tell Kubernetes you don’t need a cluster IP for your service (you do this by setting the clusterIP field to None in the service specification), the DNS server will return the pod IPs instead of the single service IP.

Instead of returning a single DNS A record, the DNS server will return multiple A records for the service, each pointing to the IP of an individual pod backing the service at that moment. Clients can therefore do a simple DNS A record lookup and get the IPs of all the pods that are part of the service. The client can then use that information to connect to one, many, or all of them.

## Troubleshooting services
Services are a crucial Kubernetes concept and the source of frustration for many developers. I’ve seen many developers lose heaps of time figuring out why they can’t connect to their pods through the service IP or FQDN. For this reason, a short look at how to troubleshoot services is in order.
When you’re unable to access your pods through the service, you should start by going through the following list:
* First, make sure you’re connecting to the service’s cluster IP from within the cluster, not from the outside.
* Don’t bother pinging the service IP to figure out if the service is accessible (remember, the service’s cluster IP is a virtual IP and pinging it will never work).
* If you’ve defined a readiness probe, make sure it’s succeeding; otherwise the pod won’t be part of the service.
* To confirm that a pod is part of the service, examine the corresponding End- points object with kubectl get endpoints.
* If you’re trying to access the service through its FQDN or a part of it (for exam- ple, myservice.mynamespace.svc.cluster.local or myservice.mynamespace) and it doesn’t work, see if you can access it using its cluster IP instead of the FQDN.
* Check whether you’re connecting to the port exposed by the service and not the target port.
* Try connecting to the pod IP directly to confirm your pod is accepting connec- tions on the correct port.
* If you can’t even access your app through the pod’s IP, make sure your app isn’t only binding to localhost.

# Volumns
Kubernetes provides this by defining storage volumes. They aren’t top-level resources like pods, but are instead defined as a part of a pod and share the same lifecycle as the pod. This means a volume is created when the pod is started and is destroyed when the pod is deleted. Because of this, a volume’s contents will persist across container restarts. After a container is restarted, the new container can see all the files that were written to the volume by the previous container. Also, if a pod contains multiple con- tainers, the volume can be used by all of them at once.

A volume is bound to the lifecycle of a pod and will stay in existence only while the pod exists, but depending on the volume type, the volume’s files may remain intact even after the pod and volume disappear, and can later be mounted into a new vol- ume. Let’s see what types of volumes exist.

Here’s a list of several of the available volume types:
* emptyDir—A simple empty directory used for storing transient data.
* hostPath—Used for mounting directories from the worker node’s filesystem
*nto the pod.
* gitRepo—A volume initialized by checking out the contents of a Git repository.
```
hostPath volumes are the first type of persistent storage we’re introducing, because both the gitRepo and emptyDir volumes’ contents get deleted when a pod is torn down, whereas a hostPath volume’s contents don’t. If a pod is deleted and the next pod uses a hostPath volume pointing to the same path on the host, the new pod will see whatever was left behind by the previous pod, but only if it’s scheduled to the same node as the first pod. Remember to use hostPath volumes only if you need to read or write sys-
tem files on the node. Never use them to persist data across pods.
```
* nfs—An NFS share mounted into the pod.
* gcePersistentDisk (Google Compute Engine Persistent Disk), awsElastic-lockStore (Amazon Web Services Elastic Block Store Volume), azureDisk (Microsoft Azure Disk Volume)—Used for mounting cloud provider-specific storage.
* cinder, cephfs, iscsi, flocker, glusterfs, quobyte, rbd, flexVolume, vsphere- Volume, photonPersistentDisk, scaleIO—Used for mounting other types of network storage.
* configMap, secret, downwardAPI—Special types of volumes used to expose cer- tain Kubernetes resources and cluster information to the pod.
* persistentVolumeClaim—A way to use a pre- or dynamically provisioned per- sistent storage. (We’ll talk about them in the last section of this chapter.)

## Using persistent storage
When an application running in a pod needs to persist data to disk and have that same data available even when the pod is rescheduled to another node, you can’t use any of the volume types we’ve mentioned so far. Because this data needs to be accessi- ble from any cluster node, it must be stored on some type of network-attached stor- age (NAS).

## Decoupling pods from the underlying storage technology
All the persistent volume types we’ve explored so far have required the developer of the pod to have knowledge of the actual network storage infrastructure available in the clus- ter. For example, to create a NFS-backed volume, the developer has to know the actual server the NFS export is located on. This is against the basic idea of Kubernetes, which aims to hide the actual infrastructure from both the application and its developer, leav- ing them free from worrying about the specifics of the infrastructure and making apps portable across a wide array of cloud providers and on-premises datacenters.

Ideally, a developer deploying their apps on Kubernetes should never have to know what kind of storage technology is used underneath, the same way they don’t have to know what type of physical servers are being used to run their pods. Infrastruc- ture-related dealings should be the sole domain of the cluster administrator.

When a developer needs a certain amount of persistent storage for their applica- tion, they can request it from Kubernetes, the same way they can request CPU, mem- ory, and other resources when creating a pod. The system administrator can configure the cluster so it can give the apps what they request.

### PersistentVolumes and PersistentVolumeClaims
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_pvc.png" width="700" height="500">
</div>

Instead of the developer adding a technology-specific volume to their pod, it’s the cluster administrator who sets up the underlying storage and then registers it in Kubernetes by creating a PersistentVolume resource through the Kubernetes API server. When creating the PersistentVolume, the admin specifies its size and the access modes it supports.

When a cluster user needs to use persistent storage in one of their pods, they first create a PersistentVolumeClaim manifest, specifying the minimum size and the access mode they require. The user then submits the PersistentVolumeClaim manifest to the Kubernetes API server, and Kubernetes finds the appropriate PersistentVolume and binds the volume to the claim.

The PersistentVolumeClaim can then be used as one of the volumes inside a pod. Other users cannot use the same PersistentVolume until it has been released by deletng the bound PersistentVolumeClaim.

We’ve already said that PersistentVolume resources are cluster-scoped and thus cannot be created in a specific namespace, but PersistentVolumeClaims can only be created in a specific namespace. They can then only be used by pods in the same namespace.

[**Difference between PV and PVC**](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_pvc_pv.png)

Consider how using this indirect method of obtaining storage from the infrastructure is much simpler for the application developer (or cluster user). Yes, it does require the additional steps of creating the PersistentVolume and the PersistentVolumeClaim, but the developer doesn’t have to know anything about the actual storage technology used underneath.
Additionally, the same pod and claim manifests can now be used on many different Kubernetes clusters, because they don’t refer to anything infrastructure-specific. The claim states, “I need x amount of storage and I need to be able to read and write to it by a single client at once,” and then the pod references the claim by name in one of its volumes.

### Dynamic provisioning of PersistentVolumes
The cluster admin can create multiple storage classes with different performance or other characteristics. The developer then decides which one is most appropriate for each claim they create.
The nice thing about StorageClasses is the fact that claims refer to them by name. The PVC definitions are therefore portable across different clusters, as long as the StorageClass names are the same across all of them.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_pv_s.png" width="700" height="500">
</div>

# ConfigMaps and Secrets: configuring applications
Regardless if you’re using a ConfigMap to store configuration data or not, you can configure your apps by
* Passing command-line arguments to containers
* Setting custom environment variables for each container
* Mounting configuration files into containers through a special type of volume

## Using Secrets to pass sensitive data to containers
To store and distribute such information, Kubernetes provides a separate object called a Secret. Secrets are much like ConfigMaps—they’re also maps that hold key-value pairs. They can be used the same way as a ConfigMap. You can
* Pass Secret entries to the container as environment variables
* Expose Secret entries as files in a volume

Kubernetes helps keep your Secrets safe by making sure each Secret is only distributed to the nodes that run the pods that need access to the Secret. Also, on the nodes themselves, Secrets are always stored in memory and never written to physical storage, which would require wiping the disks after deleting the Secrets from them.

# Accessing pod metadata and other resources from applications

## Passing metadata through the Downward API
In the previous chapter you saw how you can pass configuration data to your appli- cations through environment variables or through configMap and secret volumes. This works well for data that you set yourself and that is known before the pod is scheduled to a node and run there. But what about data that isn’t known up until that point—such as the pod’s IP, the host node’s name, or even the pod’s own name (when the name is generated; for example, when the pod is created by a ReplicaSet or similar controller)? And what about data that’s already specified elsewhere, such as a pod’s labels and annotations? You don’t want to repeat the same information in multiple places.

Both these problems are solved by the Kubernetes Downward API. It allows you to pass metadata about the pod and its environment through environment variables or files (in a downwardAPI volume). Don’t be confused by the name. The Downward API isn’t like a REST endpoint that your app needs to hit so it can get the data. It’s a way of having environment variables or files populated with values from the pod’s specifica- tion or status

## Talking to the Kubernetes API server
We’ve seen how the Downward API provides a simple way to pass certain pod and con- tainer metadata to the process running inside them. It only exposes the pod’s own metadata and a subset of all of the pod’s data. But sometimes your app will need to know more about other pods and even other resources defined in your cluster. The Downward API doesn’t help in those cases.

We can not call api server directly because of https, we can call it by kubectl proxy. The kubectl proxy command runs a proxy server that accepts HTTP connections on your local machine and proxies them to the API server while taking care of authenti- cation, so you don’t need to pass the authentication token in every request. It also makes sure you’re talking to the actual API server and not a man in the middle (by verifying the server’s certificate on each request).

### Talking to the API server from within a pod
Therefore, to talk to the API server from inside a pod, you need to take care of three things:
* Find the location of the API server.
* Make sure you’re talking to the API server and not something impersonating it. 
* Authenticate with the server; otherwise it won’t let you see or do anything.
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_download_api.png" width="700" height="500">
</div>

# StatefulSets: deploying replicated stateful applications
ReplicaSets create multiple pod replicas from a single pod template. These replicas don’t differ from each other, apart from their name and IP address. If the pod template includes a volume, which refers to a specific PersistentVolumeClaim, all replicas of the ReplicaSet will use the exact same PersistentVolumeClaim and therefore the same PersistentVolume bound by the claim.
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_stateless.png" width="700" height="500">
</div>

Because the reference to the claim is in the pod template, which is used to stamp out multiple pod replicas, you can’t make each replica use its own separate Persistent- VolumeClaim. You can’t use a ReplicaSet to run a distributed data store, where each instance needs its own separate storage—at least not by using a single ReplicaSet. To be honest, none of the API objects you’ve seen so far make running such a data store possible. You need something else.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_stateful.png" width="700" height="500">
</div>

删除stateful的pod不会导致volumn一起被删, 需要手动删除


How can a pod discover its peers without talking to the API? Is there an existing, well-known technology you can use that makes this possible? How about the Domain Name System (DNS)? 
# Kubectl

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_kubectl.png" width="700" height="500">
</div>
