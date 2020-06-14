# End-to-End ESP-Kubernetes (ESP-Kube) Architecture

In this section, we discuss how ESP servers are deployed in a Kubernetes cluster, leveraging the features of Kubernetes to create a highly available distributed system: scalability, elasticity, high availability with replication and fault tolerance, security, and portability. We refer to this environment as “ESP-Kube Architecture.”

Kubernetes has a set of building blocks that provision mechanisms to deploy, manage, and scale applications based on CPU utilization, memory usage, and other customized metrics. Kubernetes defines resources as objects. Some of the key objects are Services, Volumes, Namespaces, ConfigMaps and Secrets, Deployments, and Replica Sets. We assume readers are aware of these terms. Read more about [Kubernetes Objects here](https://kubernetes.io/docs/concepts/).

* [ESP-Kube Architecture](EndToEndESPKubeArchitecture.md#esp-kube-architecture)
    * [Namespaces at the Master Node](EndToEndESPKubeArchitecture.md#namespaces-at-the-master-node)
    * [Container Registry](EndToEndESPKubeArchitecture.md#container-registry)
    * [Worker Nodes](EndToEndESPKubeArchitecture.md#worker-nodes)


## ESP-Kube Architecture

We will now discuss the high-level end-to-end ESP-Kube multi-tenancy architecture, which demonstrates how SAS Event Stream Processing components are deployed and used, leveraging all the capabilities of Kubernetes. The architecture is independent of the underlying platform. This means that the architecture can be implemented in any public cloud environment, such as Amazon AWS, Microsoft Azure and others, private cloud or on-premises.

The Figure below depicts the multi-tenant ESP-Kube architecture of the Kubernetes master node, which consists of shared and non-shared namespaces. 

![ESP-Kube Master Node Architecture](archImages/ESPKubeMasterNode.png)

The ESP-Kube architecture relies on namespaces to provide cluster sharing among many clients, i.e., creating an independent workspace for each client with no or little sharing. In other words, multi-tenancy, a deployment architecture, is designed to provide all tenants with a dedicated share of the cluster, including storage, CPU and memory resources, configurations, user-management, and other services. Multi-tenancy can save costs while providing isolation, security, and privacy for customers running their own applications. 

Kubernetes creates namespaces by partitioning the available resources into several non-overlapping workspaces. These workspaces enable multiple clients to have their own projects, deployments, even development, test and production environments that are separate, but on the same cluster with its capabilities.

Many resources, such as application pods and services are namespaced, that is, not sharable with other namespaces, while some, such as nodes, databases with different schemas for each client, message queues, or certificate managers, are cluster-wide and not namespaced. The administrator of the ESP-Kube cluster will be responsible for the management and creation of all the non-shared and shared namespaces, setup of access controls, resource quota assignment, and environment configuration.

### Namespaces at the Master Node

We will discuss all the namespaces of the cluster in detail now.

#### NAMESPACE_OF_A_TENANT

Each tenant has its own namespace where it can deploy the ESP operator, metering server, ESP server, metering agent, persistent volume and its claim, and clients like SAS Event Stream Manager, SAS Event Stream Processing Streamviewer, and SAS Event Stream Processing Studio. A tenant can also set up tenant-specific User Account & Authorization (UAA) using tokens for authenticating users.

The ESP operator is the heart of the tenant’s namespace. It is responsible for the creation, deletion, modification, updates, assignment of resources and scaling of the SAS Event Stream Processing projects running in pods, executing the ESP server container on the worker nodes. Each ESP server container can have one and only one project; therefore, every pod contains only one ESP server container. Tenants can use SAS Event Stream Processing Studio to create their projects, which are then deployed by the ESP operator in pods.

The Namespace can also deploy services for enabling network access to the pods running in the cluster. You have several options for providing network access, such as ClusterIP, NodePort (services are accessible on a static port on each node of the cluster), LoadBalancer (cloud provider load balancers can be leveraged for automatic request routing) and Ingress. We consider ClusterIP and Ingress in our deployment.  

1.	*ClusterIP*: It creates a single IP address to provides accessibility to the pods from inside the cluster. It is useful for exposing the microservices running in the same cluster to each other. 

2.	*Ingress*: It exposes HTTP and HTTPS routes from outside of the cluster to services within the cluster. It redirects the REST/HTTP/s calls to the specific services. Ingress needs an ingress controller to satisfy Ingress (Resources) and is always implemented by a third-party proxy.

#### NAMESPACE_LOAD_BALANCER

This namespace is used to deploy an ingress controller that is shared by all the tenants. An ingress controller reads the ingress resources information to perform the routing and processing of the incoming requests according to the set of rules that are defined.  

There are many types of [ingress controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/), such as GCE ingress controller, [AWS ELB Ingress Controller](https://github.com/kubernetes-sigs/aws-alb-ingress-controller), Envoy, Traefik, Nginx, etc. We use [Nginx Ingress Controller](https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/installation.md).

The Nginx ingress controller relies on the classic load balancer (ELB) by exposing `Service=LoadBalancer` when initialized on AWS.

It is worth noting that the ingress controller does not eliminate the need for an external load balancer but only adds another layer of routing and controlling to the load balancer. An ingress controller, in other words, sits between the external load balancer and the services. The ingress controller is well integrated with Kubernetes and is simple to use and configure.  

#### NAMESPACE_CERT_MANAGER

We create this namespace to automatically provision SSL certificates for services. It uses [CustomResourceDefinitions (CRDs)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) to request certificates and configure authorities like [Issuers](https://docs.cert-manager.io/en/latest/reference/issuers.html) or [ClusterIssuers](https://docs.cert-manager.io/en/latest/reference/clusterissuers.html) to create certificates. We use cert_manager to provide TLS certificates to SAS Event Stream Processing projects and web clients. Whenever there is an incoming HTTP client request, ingress communicates with the cert_manager certificate Issuer, [Let’s Encrypt](https://letsencrypt.org/) in our case, to get a new TLS certificate.

#### NAMESPACE_KUBE_SYSTEM

This namespace contains all the components of Kubernetes which manage and control the cluster. We have discussed all these components previously in the Kubernetes Architecture section.

#### NAMESPACE_SHARED_BY_TENANT

This namespace includes services to be shared among all the clients, such as the following:

1.	Messages Bus: Kafka, SQS, Kinesis, MQTT, RabbitMQ, etc. Each tenant will have its own queues from which to read and write data. 
2.	Database: Amazon RDS, NoSQL, PostgreSQL, Key-Value, etc. Depending on the architecture, tenants can have their own dedicated schemas in the database. 
3.	ESP Services: SAS Event Stream Manager, SAS Event Stream Processing Streamviewer, and SAS Event Stream Processing Studio can also be part of the shared namespace.

### Container Registry

The Container Registry is a repository from where clients can pull the SAS ESP images, such as ESP Operator, ESP Metering Server, ESP Server, etc. Clients are free to customize the pulled ESP Server docker images according to their project needs.  

### Worker Nodes

The Figure below demonstrates the architecture of a worker node that runs three pods with different numbers of containers. This means that each pod can be configured to run different types of workloads. Here, all the pods have a common container (Container 1). The inclusion of Container 2 and Container 3 depends on the requirement for executing the workloads.

![ESP-Kube Worker Node Architecture](archImages/ESPKubeWorkerNode.png)

When the pods are deployed by the master node, the custom resource specification has information about the containers running in it.




