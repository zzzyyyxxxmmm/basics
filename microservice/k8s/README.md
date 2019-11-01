# Overview
[seven layers](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/k8s_arch.png)

* **The Kubernetes API Server**, which you and the other Control Plane components communicate with.
* **The Scheduler**, which schedules your apps (assigns a worker node to each deploy- able component of your application)
* **The Controller Manager**, which performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on
* **etcd**, a reliable distributed data store that persistently stores the cluster configuration.