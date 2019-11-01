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

