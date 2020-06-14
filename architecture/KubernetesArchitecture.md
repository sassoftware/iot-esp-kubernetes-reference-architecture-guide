# Kubernetes Architecture

Kubernetes is a software platform that allows easy deployment, configuration, automation, and management of containerized stateful and stateless applications. Kubernetes provides:

1.	Load balancing and scheduling of containers
2.	Creation, deletion, and movement of containers
3.	Easy scalability and elasticity of containers on-demand
4.	High availablility - fault tolerance and recovery with zero downtime
5.	Monitoring, service discovery and self-healing abilities

Kubernetes enables running software applications on thousands of machines across data centers, abstracting the underlying infrastructure. The size of the cluster makes no difference.  

In this section, we will discuss:
* [Components of Kubernetes Architecture](KubernetesArchitecture.md#components-of-kubernetes-architecture)
    * [Kubernetes Master Node](KubernetesArchitecture.md#kubernetes-master-node)
    * [Kubernetes Worker Node](KubernetesArchitecture.md#kubernetes-worker-node)


## Components of Kubernetes Architecture

A Kubernetes cluster is a group of nodes that consists of a master node and worker nodes. Kubernetes follows a master-slave architecture. The nodes might be bare-metal servers, on-premises virtual machines or physical machines, or virtual machines on any cloud provider. We will now discuss the various components of master and worker nodes, as shown in Figure below.

![Kubernetes Architecture](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/archImages/KubernetesArchitecture.png)

### Kubernetes Master Node

By default, a single master node acts as the controller and single point of contact. For high availability, it is possible to have multiple master nodes. In case of a failure of the master node, the replicated master node takes over with zero downtime.

The master node has many responsibilities, which consist of 1) Managing the cluster; 2) Storing information about all worker nodes; 3) Monitoring the nodes and moving the workload from one node to another in case of a node failure; 4) Starting, stopping, and managing containers on different worker nodes, and more. 

The master node is responsible for managing the cluster. Essentially, it is the heart of the cluster.

A Kubernetes master node consists of different system components, which we describe below:

1.	Etcd
2.	Kube API Server (kube-apiserver)
3.	Controller Manager
4.	Scheduler
5.	Cloud Controller Manager

#### *Etcd*

Etcd is a distributed, persistent, lightweight, and reliable key-value data store used for configuration information management, service discovery, and distributed tasks of the cluster. At any given time, etcd stores information about the state of the cluster, including the number of nodes in the cluster, their locations in data centers, what pods are running and where they are deployed, and how many replicas of pods exist. Therefore, it is important to back up the etcd data.

Etcd stores all the cluster data in a distributed manner. It implements a locking mechanism within the cluster to ensure there are no conflicts between the multiple masters. 

Etcd also implements a watch feature, an event-based interface for asynchronously monitoring changes. It is a crucial feature.  The Kubernetes API server relies on this interface for notifications of changes so that it can call the appropriate business logic components to update the state from current to desired. 

#### *Kube API Server*

The API server is the first point of contact when interacting with the Kubernetes cluster using *kubectl* command-line interface. Each *kubectl* command is converted into an API call to the API server.

The API server is a key component of the master node. It provides access to both the internal and external interface to Kubernetes, using JSON over HTTP. It allows applications to interact with each other. 

The API server is the front end of the Kubernetes control plane and the only component that communicates with etcd. All other components must communicate through the API server to get the cluster state. It processes and validates the REST operations and updates the corresponding state of the API objects in etcd, while verifying that data complies with all the agreements with service details of the deployed containers.

The API server also authenticates and authorizes all the API clients in order to enable communications. A watch mechanism, similar to etcd, is implemented by the API Server to allow the scheduler and controller manager to interact with the API server.

#### *Controller Manager*

A controller manager is a process that runs and manages several core controllers’ processes to regulate the state of the cluster and perform routine tasks. 

Logically, each controller is a separate process, but for simplicity, they are compiled into a single binary and run as a single process. 

Here is a list of a few controllers and description of their respective functions [(a full list of controllers is here)](https://github.com/kubernetes/kubernetes/tree/master/pkg/controller).


1.	*Node Controller*: detects and responds when a node goes down.
2.	*Replication Controller*: handles replication and scaling by running a specified number of replicas of pods across the cluster. It also handles spinning up and transfers of pods if the node where the pods were running fails.
3.	*Service Account and Token Controller*: creates default accounts and API server access tokens to the new namespaces.
4.	*DaemonSet Controller*: runs exactly one pod on every worker node or on a subset of worker nodes.

#### *Scheduler*

The scheduler tracks the resource utilization on each worker node in the cluster and is responsible for the distribution of the workload based on the availability of requested resources, resource requirements, and other user-defined constraints and policies, such as quality of service, affinity and anti-affinity specifications, data location, and inter-workload interference. 

The scheduler reads the operational requirements of the services and schedules the workload in the cluster in a best-fit way. Therefore, at any given time, the scheduler is aware of the total resources available and the resources assigned to process the existing workload on each worker node. Its main purpose is to ensure that the workload is not scheduled without adequate resources. 

The scheduler performs two operations to find the best node to allocate the workload: 

1.	*Filtering*: finds the set of nodes that can possibly take a new pod.
2.	*Scoring*: gives a rank to each node that passed the filtering step. The node with the highest rank is selected for the new pod.

#### *Cloud Controller Manager*

Cloud Controller Manager runs several controllers that are required for interactions with cloud providers. It runs only the cloud provider-specific controller loops, which can be enabled and disabled depending on the requirement. 

The available controllers are as follows:

1.	*Node Controller*: initializes a node running in the cluster of the cloud provider with considerations of a specific zone, instance information (type and size), and more. If a node is unreachable, the controller checks node status with the cloud provider.
2.	*Route Controller*: sets up routes in the underlying cloud provider infrastructure so that the pods in Kubernetes can communicate easily (only applicable for Google Compute Engine clusters).
3.	*Service Controller*: creates, updates, and deletes cloud provider load balancers, such as Amazon ELB, Google LB, Oracle Cloud Infrastructure LB. 

These controllers are responsible for abstracting resources offered by the cloud providers and matching them to the corresponding Kubernetes objects.

### Kubernetes Worker Node

The Kubernetes worker node is also referred to as a "worker" or simply a "node". It is a machine in the Kubernetes cluster that runs the containers inside a pod and processes the incoming workload. The master node communicates with the worker nodes for deploying the pod, assigning jobs, and load balancing among nodes. It is a best practice to run all the workloads in these worker nodes, as the master node needs to manage the entire cluster configuration and ensure smooth functioning. 

Each node must run three main components:

1.	Kubelet
2.	Kube-proxy
3.	Container Runtime

These components maintain the running pods at the underlying node and provide a Kubernetes runtime environment. 

#### *Kubelet*

The kubelet component is the main agent that runs on every worker node in the cluster. It is responsible for starting, stopping, and maintaining application containers running in pods. Kubelet gets specifications from the kube-apiserver and ensures that the current state of the containers in a pod or pods matches the requirements specified (*PodSpecs*) for that pod stored in etcd. If the state of the pod does not comply with the desired state, the kubelet de-deploys the pods on the same node. Kubelet ensures that pods are always running in a healthy desired state. 

Kubelet sends a status report every few seconds via heartbeat messages from all the pods running on the node. If the master kube-apiserver detects a node failure, the replication controller launches all the pods from the failed node on the healthy node in the cluster. 

Kubelet doesn’t manage containers which are not created by the Kubernetes master node. Download [Source code of kubelet](https://github.com/kubernetes/kubernetes/tree/master/pkg/kubelet).

#### *Kube-proxy*

The kube-proxy component is an implementation of a network proxy that runs on each worker node. It manages the individual host subnetting and provides external communication. The kube-proxy keeps the networking environment seamless, accessible, and isolated. 

Based on the IP address and port numbers of incoming requests, kube-proxy forwards them to the correct pods/containers across the various isolated networks in the cluster and performs basic load balancing.  It can handle simple or round-robin TCP, UDP, and SCTP request forwarding. 

Each worker node interacts with Kubernetes services only via kube-proxy. 

#### *Container Runtime*

The container runtime is a software component that is responsible for running containers inside a pod. To spin up a container and run it inside a pod, Kubernetes relies on the container runtime. Kubelet interacts with the runtime to perform on-demand start, stop, and deletion of the containers on the underlying worker node. 

Kubernetes supports any [Open Container Initiative (OCI) compliant container runtimes](https://github.com/opencontainers/runtime-spec), such as Docker, rkt, containerd, and any implementation of Kubernetes Container Runtime Interface (CRI).

#### *Pod*

A Kubernetes pod is a single container, or a group of containers deployed together on the same host sharing the storage and computational resources, unique network IP address and port, and configurations that manage how the containers must run. A pod is the basic scheduling unit in Kubernetes, with the following characteristics:

*Shared Namespace*: All the containers in a pod are connected and interact with each other while maintaining a degree of isolation from others. They all belong to the same namespace. This brings easy intra-pod communication and management, and flexibility. 

*Shared Volumes*: A storage volume could be attached to one or many containers in the pod. Many types of volumes are supported by Kubernetes. We will discuss them later. 

*Shared Resources*: Pods also must share resource limits, such as CPU and memory. 

