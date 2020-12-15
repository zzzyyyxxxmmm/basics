# C1

organization -> folder -> project

## Compute Resources

1. Virtual Machines
2. Managed Kubernetes Clusters
3. Serverless Computing
Google Cloud Platform has two serverless computing options: App Engine and Cloud Functions. App Engine is used for applications and containers that run for extended periods of time, such as a website backend, point-of-sale system, or custom business appli- cation. Cloud Functions is a platform for running code in response to an event, such as uploading a file or adding a message to a message queue. This serverless option works well when you need to respond to an event by running a short process coded in a function or by calling a longer-running application that might be running on a VM, managed cluster, or App Engine.

## Storage
1. Object Storage
2. File Storage
3. Block Storage
4. Caches

Object storage is designed for highly reliable and durable storage of objects, such as images or data sets. Object storage has more limited functionality than file system–based storage systems. File system–based storage provides hierarchical directory storage for files and supports common operating system and file system functions. File system services provide network-accessible file systems that can be accessed by multiple servers. Block storage is used for storing data on disks. File systems and databases make use of block storage systems. Block storage is used with persistent storage devices, such as SSDs and HDDs. Caches are in-memory data stores used to minimize the latency of retrieving data. They do not provide persistent stor- age and should never be considered a “system of truth.”

## Cloud Computing vs. Data Center Computing
1. Rent Instead of Own Resources
2. Pay-as-You-Go-for-What-You-Use Model
3. Elastic Resource Allocation
4. Specialized Services

# Computing Components of Google Cloud Platform
 
* Computing resources
* Storage resources
* Databases
* Networking services
* Identity management and security Development tools
* Management tools Specialized services

## Computing Resources

### Compute Engine
Compute Engine is a service that allows users to create VMs, attach persistent storage to those VMs, and make use of other GCP services, such as Cloud Storage.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/gcp_1.png" width="500" height="700">
</div>

### Kubernetes Engine

### App Engine
App Engine is GCP’s compute PaaS offering. With App Engine, developers and application administrators don’t need to concern themselves with configuring VMs or specifying Kubernetes clusters. Instead, developers create applications in a popular programming lan- guage such as Java, Go, Python, or Node.js and deploy that code to a serverless application environment.
App Engine manages the underlying computing and network infrastructure. There is no need to configure VMs or harden networks to protect your application. App Engine is well suited for web and mobile backend applications.
App Engine is available in two types: standard and flexible.

In the standard environment, you run applications in a language-specific sandbox, so your application is isolated from the underlying server’s operating system as well as from other applications running on that server. The standard environment is well suited to applications that are written in one of the supported languages and do not need operating system packages or other compiled software that would have to be installed along with the application code.

In the flexible environment, you run Docker containers in the App Engine environment. The flexible environment works well in cases where you have application code but also need libraries or other third-party software installed. As the name implies, the flexible environ- ment gives you more options, including the ability to work with background processes and write to local disk.

### Cloud Functions
Google Cloud Functions is a lightweight computing option that is well suited to event-driven processing. Cloud Functions runs code in response to an event, like a file being uploaded to Cloud Storage or a message being written to a message queue. The code that executes in the Cloud Functions environment must be short-running—this computing service is not designed to execute long-running code. If you need to support long-running applications or jobs, consider Compute Engine, Kubernetes Engine, or App Engine.

Cloud Functions is often used to call other services, such as third-party application programming interfaces (APIs) or other GCP services, like a natural language translation service.
Like App Engine, Cloud Functions is a serverless product. Users only need to supply code; they do not need to configure VMs or create containers. Cloud Functions will auto- matically scale as load increases.

## Storage Components of Google Cloud Platform

### Cloud Storage
Cloud Storage is GCP’s object storage system. Objects can be any type of file or binary large object. Objects are organized into buckets, which are analogous to directories in a file system. It is important to remember that Cloud Storage is not a file system. It is a ser- vice that receives, stores, and retrieves files or objects from a distributed storage system. Cloud Storage is not part of a VM in the way an attached persistent disk is. Cloud Storage is accessible from VM (or any other network device with appropriate privileges) and so complements file systems on persistent disks.

Each stored object is uniquely addressable by a URL. For example, a .pdf version of this chapter, called chapter1.pdf, that is stored in a bucket named ace-certification-exam-prep would be addressable as follows:
```https://storage.cloud.google.com/ace-certification-exam-prep/chapter1.pdf```

Cloud Storage is useful for storing objects that are treated as single units of data. For example, an image file is a good candidate for object storage. Images are generally read and written all at once. There is rarely a need to retrieve only a portion of the image. In general, if you write or retrieve an object all at once and you need to store it independently of serv- ers that may or may not be running at any time, then Cloud Storage is a good option.

Regional storage keeps copies of objects in a single Google Cloud region. Multi-region is also allowed.

A useful feature of Cloud Storage is the set of lifecycle management policies that can automatically manage objects based on policies you define.

### Persistent Disk
Persistent disks are storage service that are attached to VMs in Compute Engine or Kubernetes Engine. Persistent disks provide block storage on solid-state drives (SSDs) and hard disk drives (HDDs). SSDs are often used for low-latency applications where persistent disk performance is important. SSDs cost more than HDDs, so applications that require large amounts of persistent disk storage but can tolerate longer read and write times can use HDDs to meet their storage requirements.

An advantage of persistent disks on the Google Cloud Platform is that these disks sup- port multiple readers without a degradation in performance. This allows for multiple instances to read a single copy of data. Disks can also be resized as needed while in use without the need to restart your VMs.

Persistent disks can be up to 64TB in size using either SSDs or HDDs.

### Cloud Storage for Firebase
Mobile app developers may find Cloud Storage for Firebase to be the best combination of cloud object storage and the ability to support uploads and downloads from mobile devices with sometimes unreliable network connections.
The Cloud Storage for Firebase API is designed to provide secure transmission as well as robust recovery mechanisms to handle potentially problematic network quality. Once files, like photos or music recordings, are uploaded into Cloud Storage, you can access those files through the Cloud Storage command-line interface and software development kits (SDKs).

### Cloud Filestore
Sometimes, developers need to have access to a file system housed on network-attached storage. For these use cases, the Cloud Filestore service provides a shared file system for use with Compute Engine and Kubernetes Engine.

Filestore can provide high numbers of input-output operations per second (IOPS) as well as variable storage capacity. File system administrators can configure Cloud Filestore to meet their specific IOPS and capacity requirements.

Filestore implements the Network File System (NFS) protocol so system administrators can easily mount shared file systems on virtual servers.

Storage systems like the ones just described are used to store coarse-grained objects, like files. When data is more finely structured and has to be retrieved using query languages that describe the subset of data to return, then it is best to use a database management system.

### Databases

### Cloud SQL

### Cloud Bigtable
Cloud Bigtable is designed for petabyte-scale applications that can manage up to billions of rows and thousands of columns. It is based on a NoSQL model known as a wide-column data model, and unlike Cloud SQL that supports relational databases. Bigtable is suited for applications that require low-latency write and read operations. It is designed to support millions of operations per second.
Bigtable integrates with other Google Cloud services, such as Cloud Storage, Cloud Pub/Sub, Cloud Dataflow, and Cloud Dataproc. It also supports the Hbase API, which is an API for data access in the Hadoop big data ecosystem. Bigtable also integrates with open source tools for data processing, graph analysis, and time-series analysis.

### Cloud Spanner
Cloud Spanner is Google’s globally distributed relational database that combines the key benefits of relational databases, such as strong consistency and transactions, with the abil- ity to scale horizontally like a NoSQL database. Spanner is a high availability database with a 99.999 percent availability Service Level Agreements (SLA), making it a good option for enterprise applications that demand scalable, highly available relational database services.
Cloud Spanner also has enterprise-grade security with encryption at rest and encryption in transit, along with identity-based access controls.
Cloud Spanner supports ANSI 2011 standard SQL.

### Cloud Datastore
Cloud Datastore is a NoSQL document database. This kind of database uses the concept of a document, or collection of key-value pairs, as the basic building block. Documents allow for flexible schemas. For example, a document about a book may have key-value pairs list- ing author, title, and date of publication. Some books may also have information about companion websites and translations into other languages. The set of keys that may be included does not have to be defined prior to use in document databases. This is especially helpful when applications must accommodate a range of attributes, some of which may not be known at design time.
Cloud Datastore is accessed via a REST API that can be used from applications running in Compute Engine, Kubernetes Engine, or App Engine. This database will scale automati- cally based on load. It will also shard, or partition, data as needed to maintain perfor- mance. Since Cloud Datastore is a managed service, it takes care of replication, backups, and other database administration tasks.
Although it is a NoSQL database, Cloud Datastore supports transactions, indexes, and SQL-like queries.
Cloud Datastore is well suited to applications that demand high scalability and struc- tured data and do not always need strong consistency when reading data. Product catalogs, user profiles, and user navigation history are examples of the kinds of applications that use Cloud Datastore.

### Cloud Memorystore
Cloud Memorystore is an in-memory cache service. Other databases offered in GCP
are designed to store large volumes of data and support complex queries, but Cloud Memorystore is a managed Redis service for caching frequently used data in memory. Caches like this are used to reduce the time needed to read data into an application. Cloud Memorystore is designed to provide submillisecond access to data.
As a managed service, Cloud Memorystore allows users to specify the size of a cache while leaving administration tasks to Google. GCP ensures high availability, patching, and automatic failover so users don’t have to.

### Cloud Firestore
Cloud Firestore is another GCP-managed NoSQL database service designed as a **back- end for highly scalable web and mobile applications.** A distinguishing feature of Cloud Firestore is its client libraries that provide offline support, synchronization, and other fea- tures for managing data across **mobile devices, IoT devices**, and backend data stores. For example, applications on mobile devices can be updated in real time as data in the back- end changes.
Cloud Firebase includes a Datastore mode, which enables applications written for Datastore to work with Cloud Firebase as well. When running in Native mode, Cloud Firestore provides real-time data synchronization and offline support.

## Networking Components of Google Cloud Platform

### Networking Services
1. Virtual Private Cloud
2. Cloud Load Balancing
3. Cloud Armor
4. Cloud CDN With content delivery networks (CDNs), users anywhere can request content from systems distributed in various regions. 
5. Cloud Interconnect Cloud Interconnect is a set of GCP services for connecting your existing networks to the Google network. 
6. Cloud DNS

# Projects, Service Accounts, and Billing

# Introduction to Computing in Google Cloud

# Computing with Compute Engine Virtual Machines

# APP ENGINE
App Engine Standard applications consist of four components:
* Application
* Service 
* Version 
* Instance

# Cloud Function
There are some terms you need to know before going any further into Cloud Functions:
* Events
* Triggers 
* Functions

Events are a particular action that happens in Google Cloud, such as a file is uploaded to Cloud Storage or a message (called a topic) is written to a Pub/Sub message queue. There are different kinds of actions associated with each of the events. Currently, GCP supports events in five categories:

* Cloud Storage 
* Cloud 
* Pub/Sub 
* HTTP
* Firebase
* Stackdriver Logging

Events in Cloud Storage include uploading, deleting, and archiving a file. Cloud Pub/Sub has an event for publishing a message. The HTTP type of event allows developers to invoke a function by making an HTTP request using POST, GET, PUT, DELETE, and OPTIONS calls. Firebase events are actions taken in the Firebase database, such as database triggers, remote configuration triggers, and authentication triggers. You can set up a function to respond to a change in Stackdriver Logging by forwarding log entries to a Pub/Sub topic and triggering a response from there.
For each of the Cloud Functions–enabled events that can occur, you can define a trigger. A trigger is a way of responding to an event.
Triggers have an associated function. The function is passed arguments with data about the event. The function executes in response to the event.

# 3 Projects, Service Accouts, and Billing
* Organization 
* Folder
* Project

For example, you may have an application that needs to access a database, but you do not want to allow users of the application to access the database directly. Instead, all user requests to the database should go through the application. A service account can be created that has access to the database. That service account can be assigned to the application so the application can execute queries on behalf of users without having to grant database access to those users.

Large enterprises should use invoicing when incurring large charges, which makes option D the right answer. A self-service account is appropriate only for amounts that are within the credit limits of credit cards. 

When a service account is created, Google generates encrypted keys for authentication,

# 4 Introduction to Computing in Gooogle Cloud
App Engine is a PaaS compute service that provides a managed platform for running applications. When you use App Engine, your focus is on your application and not on the VMs that run the application. Instead of configuring VMs, you specify some basic resource requirements along with your application code, and Google will manage the resources needed to run the code. This means that App Engine users have less to manage, but they also have less control over the compute resources that are used to execute the application.

Like VM instances, applications in App Engine are created within a project.


The standard environment is the original App Engine environment. It consists of a preconfigured, language-specific runtime. There are currently two generations of the standard environment. The second generation improves on the performance of the first generation and has fewer limitations.

Currently, App Engine standard environment users can choose from the following supported languages:
First Generation

* Python 2.7
* PHP 5.5
* Go 1.9

Second Generation

* Java 8
* Python 3.7 (beta)
* PHP 7.2 (beta)
* Node.js 8 (beta) and 10 (beta)
* Go 1.11 (beta)

App Engine Flexible Environment: Manage container, more like kubernetes

The only states a Kubernetes deployment can be in are progressing, completed, and failed.

# 5 Computing with Compute Engine Virtual Machines
set up billing -> create proect -> create instances

gcloud compute instances create --boot-disk-type
gcloud compute project_info describe

The two operations you can specify when using the book disk configuration are adding a new disk and attaching an existing disk,

# 6 Managing Virtual Machines
gcloud compute instances  start stop delete list

Instance groups are sets of VMs that are managed as a single entity. Any gcloud or console command applied to an instance group is applied to all members of the instance group. Google provides two types of instance groups: managed and unmanaged.

Managed groups consist of groups of identical VMs. They are created using an instance template, which is a specification of a VM configuration, including machine type, boot disk image, zone, labels, and other properties of an instance. Managed instance groups can automatically scale the number of instances in a group and be used with load balancing to distribute workloads across the instance group. If an instance in a group crashes, it will be recreated automatically. Managed groups are the preferred type of instance group.

Unmanaged groups should be used only when you need to work with different configurations within different VMs within the group.

gcloud compute instance-templates create INSTANCE

GPU gpu library installed before, instance must be stopped while attaching GPU

# 7 Computing with Kubernetes 
gcloud container clusters create ch07-cluster --num-nodes=3 --region=us-central1

# 8 Managing Kubernetes Clusters
gcloud container clusters list

gcloud container clusters describe --zone us-central1-a standard-cluster-1

To view the status of clusters from the command line, use the gcloud container commands, but to get information about Kubernetes managed objects, like nodes, pods, and containers, use the kubectl command.

# 9 Computing with APP engine

App Engine Standard applications consist of four components:

* Application
* Service
* Version
* Instance

App Engine provides three ways to split traffic: by IP address, by HTTP cookie, and by random selection.

The command to split traffic is gcloud app services set-traffic. Here’s an example:

gcloud app services set-traffic serv1 --splits v1=.4,v2=.6

This will split traffic with 40 percent going to version 1 of the service named serv1 and 60 percent going to version 2. If no service name is specified, then all services are split.

The gcloud app services set-traffic command takes the following parameters:

■ --migrate indicates that App Engine should migrate traffic from the previous version to the new version.

■ --split-by specifies how to split traffic using either IP or cookies. Possible values are ip, cookie, and random.


A project can support only one App Engine app

App Engine applications are accessible from URLs that consist of the project name followed by appspot.com


# 10 Computing with Cloud Functions
Events: Cloud Storage, Cloud Pub/Sub, HTTP, Firebase, Stackdriver Logging

By default Cloud Functions can run for up to 1 minute before timing out, up  to 9min

Google currently supports three runtime environments:

* Python 3
* Node.js 6
* Node.js 8

### Deploying a Cloud Function for Cloud Storage Events Using gcloud Commands

* google.storage.object.finalize
* google.storage.object.delete
* google.storage.object.archive
* google.storage.object.metadataUpdate

```
gcloud functions deploy cloud_storage_function_test \
--runtime python37 \
--trigger-resource gcp-ace-exam-test-bucket \
--trigger-event google.storage.object.finalize
```


# 11 Planning Storage in the Cloud

### persistent disk
disk: standard & SSD persistent disk

#### object storage
store large volumes of data, files are stored in the system as atomic units, you can't read part of them

nearline and coldline is used to storefiles which are access less than once per month and year

version can be used to retrive history datas, 

regional and multi-reginol can be configured when it create, 不能中途修改

### relational database cloud SQL, Cloud Spanner, Big Query
big query: create data set before, it is designed for data warehousing and analytics,

Cloud Spanner is Google’s globally distributed relational database that combines the key benefits of relational databases, such as strong consistency and transactions, with the ability to scale horizontally like a NoSQL database. Spanner is a high availability database with a 99.999 percent availability Service Level Agreements (SLA), making it a good option for enterprise applications that demand scalable, highly available relational database services.
### noSQL Datastore, Cloud firestore(for mobile), BigTable
Bigtable columms多, Bigtable is designed for petabyte-scale databases. Both operational databases, like storing IoT data, and analytic processing, like data science applications, can effectively use Bigtable. This database is designed to provide consistent, low-millisecond latency. Bigtable runs in clusters and scales horizontally.



# 12 Deploying storage in google cloud platform

### mysql
```gcloud sql connect dsakjdsalk -user=root```

1. create backup ```gcloud sql backup create --async --instance```
2. auto backup ```gcloud sql instances patch sdisao -backup-start-time 12:30```

### datastore
GQL
### big query
--dry run allow to caculate the price

```bq --location=us show -j dsadjsakldjsa```

### cloud spanner

### bigtable
no sql

1. gcloud components install cbt
2. cbt createtable dasd

### dataproc
Spark PySpart SparkR

```gloud dataproc jobs submit spark --clusterdsakdj --jar ```

# 13 Loading Data into Storage

### import and export: file
1. create bucket
```
gsutil mb gs://
```

2. upload file and move
```
gsutil cp /home/dsad gs://dsadsa
gsutil mv /home/dsad gs://dsadsa
```

### import and export: cloud SQL
Export to csv and sql only

1. check permission to write to bucket```gcloud sql instances describe ace-exam-mysql1```
2. assign permission ```gsutil acl ch -u service_accout address:W gs://bucketname
3. gcloud sql export sql|csv instance_name gs://dsadsa --database=database_name

### Cloud Datastore command only
```gcloud datastore export --namespace gs://```

### import and export: bigquery
export format: csv avro json

```bq extract --destinatioon_fornat csv --compressioon GZIP 'dsadsa` gs://dsadsadsa ```

import format: csv json avro parquet orc cloud datastore backup

```bq load --autodetect --source_format=csv mysdask.table gs://```
autodetect is used to detect the schema

### cloud spanner
dataflow

### big table
no console to do that, you have two options: java or HBase interface

### cloud dataproc
1. gcloud components install beta
2. gcloud beta dataproc clusters export asdsadak --destination=gs://
3. gcloud beta dataproc clusters import 

### stream data to cloud pub/sub
1. create topics ```gloud pubsub topics create```
2. sub ```gcloud pubsub subscroptions create --topic ```

# 14 Networking in the Cloud

gcloud compute networks create ace-exam-vpc1 --subnet-mode=custom

Firewall rules consist of several components:

■ Direction: Either ingress or egress.

■ Priority: Highest-priority rules are applied; any rule with a lower priority that matches are not applied. Priority is specified by an integer from 0 to 65535. 0 is the highest priority, and 65535 is lowest.

■ Action: Either allow or deny. Only one can be chosen.

■ Target: An instance to which the rule applies. Targets can be all instances in a network, instances with particular network tags, or instances using a specific service account.

■ Source/destination: Source applies to ingress rules and specifies source IP ranges, instances with particular network tags, or instances using a particular service account. You can also use combinations of source IP ranges and network tags and combinations of source IP ranges and service accounts used by instances. The IP address 0.0.0.0/0 indicates any IP address. The Destination parameter uses only IP ranges.

■ Protocol and port: A network protocol such as TCP, UDP, or ICMP and a port number. If no protocol is specified, then the rule applies to all protocols.

■ Enforcement status: Firewall rules are either enabled or disabled. Disabled rules are not applied even if they match. Disabling is sometimes used to troubleshoot problems with traffic getting through when it should not or not getting through when it should.

All VPCs start with two implied rules: One allows egress traffic to all destinations (IP address 0.0.0.0/0), and one denies all incoming traffic from any source (IP address 0.0.0.0/0). Both implied rules have priority 65535, so you can create other rules with higher deny or allow traffic as you need. You cannot delete an implied rule.

When a VPC is automatically created, the default network is created with four network rules. These rules allow the following:

■ Incoming traffic from any VM instance on the same network

■ Incoming TCP traffic on port 22, allowing SSH

■ Incoming TCP traffic on port 3389, allowing Microsoft Remote Desktop Protocol (RDP)

■ Incoming Internet Control Message Protocol (ICMP) from any source on the network

The default rules all have priority 65534.

oganization folder

0-65535lowest

The autonomous system number (ASN) is a number used to identify a cloud router on a network,

VPN在Hybrid里创建


* You are using gcloud to create a firewall rule. Which parameter would you use to specify the subnet it should apply to?

A. ––subnet

B. **––network **

C. ––destination

D. ––source-ranges



* You are using Cloud Console to create a VPN. You want to configure the GCP end of the VPN. What section of the Create VPN form would you use?

A. Tunnels

B. Routing Options

C. Google Compute Engine VPN

D. IKE Version

C. The Google Compute Engine VPN is where you specify information about the Google Cloud end of the VPN connection, so option C is correct. You specify name, descrip- tion, network, region, and IP address. Option A is incorrect because tunnels are about the connections between the cloud and the remote network. Option B is incorrect; Routing Options is about how to configure routers. Option D is incorrect; IKE Version is about exchanging secret keys.

# 15 Networking in the Cloud: DNS, Load Balancing, and IP addressing

for example, an A record maps a hostname to IP addresses in IPv4. AAAA records are used in IPv6 to map names to IPv6 addresses. CNAME records hold the canonical name, which contains alias names of a domain.

Types of Load Balancers

Load balancers can distribute load within a single region or across multiple regions. The several load balancers offered by GCP are characterized by three features:

■ Global versus regional load balancing

■ External versus internal load balancing

■ Traffic type, such as HTTP and TCP 368

Global load balancers are used when an application is globally distributed. Regional load balancers are used when resources providing an application are in a single region. There are three global load balancers:

■ HTTP(S), which balances HTTP and HTTPS load across a set of backend instances

■ SSL Proxy, which terminates SSL/TLS connections, which are secure socket layer connections. This type is used for non-HTTPS traffic.

■ TCP Proxy, which terminates TCP sessions at the load balancer and then forwards traffic to backend servers.

The regional load balancers are as follows:

■ Internal TCP/UDP, which balances TCP/UDP traffic on private networks hosting internal VMs

■ Network TCP/UDP, which enables balancing based on IP protocol, address, and port. This load balancer is used for SSL and TCP traffic not supported by the SSL Proxy and TCP Proxy load balancers, respectively.

External load balancers distribute traffic from the Internet, while internal load balancers distribute traffic that originates within GCP. The Internal TCP/UDP load balancer is the only internal load balancer. The HTTP(S), SSL Proxy, TCP Proxy, and Network TCP/UDP load balancers are all external.

You will need to consider the traffic type too when choosing a load balancer. HTTP and HTTPS traffic needs to use external global load balancing. TCP traffic can use external global, external regional, or internal regional load balancers. UDP traffic can use either external regional or internal regional load balancing.

gcloud compute networks subnets expand-ip-range ace-exam-subnet1 --prefix-length 16

gcloud beta compute addresses create ace-exam-reserved-static1 --region=us-west2 --network-tier=PREMIUM

# 16 Deployinig Applicatioons with Cloud Launcher and Deployment Manager

gcloud deployment-manager deployments create quickstart-deployment --config vm.yaml

jinja is for simple templates and python is for compliated templates

# 17 Configuring Access and Security

You can also see a list of users and roles assigned across a project using the command gcloud projects get-iam-policy. For example, to list roles assigned to users in a project with the project ID ace-exam-project, use this:

gcloud projects get-iam-policy ace-exam-project

Predefi ned roles are grouped by service. For example, App Engine has five roles:

■ App Engine Admin, which grants read, write, and modify permission to application and configuration settings. The role name used in gcloud commands is roles/appengine.appAdmin.

■ App Engine Service Admin, which grants read-only access to configuration settings and write access to module-level and version-level settings. The role name used in gcloud commands is roles/appengine.serviceAdmin.

■ App Engine Deployer, which grants read-only access to application configuration and settings and write access to create new versions. Users with only the App Engine Deployer role cannot modify or delete existing versions. The role name used in gcloud commands is roles/appengine.deployer.

■ App Engine Viewer, which grants read-only access to application configuration and settings. The role name used in gcloud commands is roles/appengine.appViewer.

■ App Engine Code Viewer, which grants read-only access to all application configurations, settings, and deployed source code. The role name used in gcloud commands is roles/appengine.codeViewer.

gcloud iam roles describe

gcloud projects add-iam-policy-binding ace-exam-project --member user:jane@ aceexam.com --role roles/appengine.deployer

gcloud compute instances set-service-account ace-instance --service-account examadmin@ace-exam-project.iam.gserviceaccount.com --scopes compute-rw,storage-ro

# Monitor Logging, and Cost Estimating
