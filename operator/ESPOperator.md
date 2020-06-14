# SAS Event Stream Processing Operator 

The Event Stream Processing Operator is the first point of contact for developing, deploying, and testing an ESP server in a Kubernetes cluster. ESP operator extensively uses the underlying features and characteristics of Kubernetes. 


* [Using the ESP Operator](ESPOperator.md#using-the-esp-operator)
    * [Deployment of ESP Operator](ESPOperator.md#deployment-of-esp-operator)
    * [Create a Deployment](ESPOperator.md#create-a-deployment)
* [Prerequisites](ESPOperator.md#prerequisites)
    * [Set up NFS](ESPOperator.md#set-up-nfs)
    * [Persistent Volume (pv) and Persistent Volume Claim (pvc)](ESPOperator.md#persistent-volume-pv-and-persistent-volume-claim-pvc)
    *  [Kubernetes Metrics Server](ESPOperator.md#kubernetes-metrics-server)
* [The Ingress](ESPOperator.md#the-ingress)
    *  [The Ingress Resource](ESPOperator.md#the-ingress-resource)
    *  [The Ingress Controller](ESPOperator.md#the-ingress-controller)
    *  [The Nginx Ingress Controller](ESPOperator.md#the-nginx-ingress-controller)
    *  [Configuration and Integration of the Ingress](ESPOperator.md#configuration-and-integration-of-the-ingress)
* [The Filebrowser](ESPOperator.md#the-filebrowser)


# Using the ESP Operator

The ESP operator primarily executes the custom resources that the ESP server can run. Custom resource embeds the definition and the specifications of the project that the ESP server runs. Whenever there is a new custom resource, the ESP operator makes sure to read it, deploy it and spin up a corresponding pod in the worker nodes. The ESP Operator also makes sure to configure the Ingress endpoints. These endpoints are used by the Nginx Ingress Controller to expose the REST service for the ESP servers running in the pods to communicate with the external world. 
We will discuss ingress later in this module.


## Deployment of ESP Operator

Download the project directory from the SAS Git repository that contains all the required tools to deploy, manage, and test the ESP servers running on the worker nodes of the Kubernetes cluster. 

The directory contains scripts to launch the ESP operator, metering service, ingress, and other configurations for the single-tenant namespace. SAS has also included examples to test the ESP operator, its functionalities with support from Kubernetes.

The following three scripts are used to launch the components of the *NAMESPACE_OF_A_TENANT*. These scripts are in the `operator/bin` folder of the repository: 

1.	`./bin/mkdeploy` – This script launches the `.yaml` configuration files located in the folder named `/templates` for the tenant’s namespace. It pulls the Docker images for the given parameters from the container registry, i.e., ESP operator, ESP Server, and metering server. It creates a `/deploy` folder that has all the manifests ready for deployment for the given namespace.
2.	`./bin/dodeploy` – This script deploys all the manifests created under the `/deploy` folder. 
3.	`./bin/mkproject` – This script is used to create manifests to be used by Kubernetes to launch pods on the worker nodes. The `.yaml` configurations that are created tell the Kubernetes `kube-apiserver` to spin up the pods with an ESP server container, which will execute the specified SAS Event Stream Processing project. It converts SAS Event Stream Processing models into custom resources for Kubernetes.

## Create a Deployment

Go to the `operator` folder under the main folder `esp-kubernetes` (in the downloaded repository).

**NOTE**: For the rest of this document, for all the sample command examples, we will use `<namespace>` as `espkube`.

### mkdeploy

The `mkdeploy` script uses the `.yaml` files located in the `/templates` folder. The files in the `/templates` folder are required to configure the underlying Kubernetes cluster environment and are used by the `mkdeploy` script. The script replaces the placeholder values with the real values of the Kubernetes cluster.

Using `mkdeploy` script:

```posh
[cloud-user@espkube-1 operator]$ ./bin/mkdeploy
Usage: ./bin/mkdeploy

     GENERAL options

          -r                          -- remove existing deploy/
                                          before creating
          -y                          -- no prompt, just execute
          -n <namespace>              -- specify K8 namespace
          -d <ingress domain root>    -- project domain root,
                                          proj.ns.<domain root>
          -l <esp license file>       -- SAS ESP license
          -c <SAS certificate file>   -- SAS_CA_Certificate.pem

     options for operator deployment

          -o <esp operator image>     -- esp operator docker image
          -s <esp server image>       -- esp server docker image
          -m <esp meter image>        -- esp metering docker image
          -a <sas meter agent image>  -- esp meter agent docker image

     options for single user client deployment

          -e <esm image>              -- esp esm docker image
          -t <esp studio image>       -- esp studio docker image
          -v <esp streamviewer image> -- esp streamviewer docker image
[cloud-user@espkube-1 operator]$
````

According to the usage instructions of the script, we provide:

1.	SAS ESP Operator image
2.	SAS ESP Server image
3.	SAS ESP Meter image
4.	SAS Meter Agent image
5.	Ingress domain root 
6.	SAS ESP License file
7.	SAS certificate

For non-SAS employees, all the images can be fetched from the [SAS secure repository to your private mirror registry](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=dplyesp0phy0lax&docsetTarget=p1waqvpevifcbvn1sjcr8d15j5m0.htm&locale=en).
The [license and certificate files](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=dplyesp0phy0lax&docsetTarget=p1bte5sqqf6cban0zm2qi61y7gul.htm&locale=en) are sent to you on your business email upon making a product order for SAS Event Stream Processing for Edge Computing.

For SAS Employees, all the images can be fetched from the [SAS Container Registry](http://docker.sas.com/repositories/pdt). To obtain the license files you can [make a product order here](https://makeorder.sas.com/makeorder/orderinfo) for product SAS Event Stream Processing for Edge Computing. 

The metering server will expose itself on the ingress host as `espmeter.<namespace>.sas.com` only for the pods that were created using the ESP Operator. The ESP Operator also exposes the REST ports on the pods for communicating with the outside world on `<servicename>.<namespace>.sas.com`

Here is an example to invoke the `mkdeploy` script from the master node:

```sh
# ./bin/mkdeploy -r -n espkube 
  -o docker.sas.com/shhuan/esp-operator:latest 
  -s docker.sas.com/pdt/sas-esp:6.2.0-20190918.1568806516268 
  -m docker.sas.com/pdt/sas-espmbs:6.2.0-20190916.1568633055007 
  -a repulpmaster.unx.sas.com/lookaside/18b072c9-60bc-45ca-a854-43c3ae8c13b7:latest 
  -d sas.com 
  -l /home/cloud-user/licenses/SASViyaV0300_9C7K7Y_70180938_Linux_x86-64.jwt 
  -c /home/cloud-user/ca-certificates/SAS_CA_Certificate.pem
````

**NOTE**:  The -l option requires a JWT license file and not a TXT file.

### dodeploy

The `./bin/dodeploy` script deploys the manifests in the `/deploy` folder, which is created by the `mkdeploy` script on the Kubernetes cluster. Before performing the deployment, it verifies that the namespace exists. In the absence of the namespace, the script prompts to create a namespace.  

```posh
[cloud-user@espkube-1 operator]$ ./bin/dodeploy
Usage: ./bin/dodeploy
          -n <namespace>             -- specify K8 namespace
[cloud-user@espkube-1 operator]$
````

The only requirement for running the `dodeploy` script is providing the namespace. 

Here is an example showing how to use the script:

```sh
# ./bin/dodeploy -n espkube 
````

After the deployment, you would see the ESP operator, SAS Event Stream Processing Meter and SAS Event Stream Processing Metering Agent pods running in the `espkube` namespace.

```posh
[cloud-user@espkube-1 operator]$ kubectl get pods -n espkube
NAME                                                 READY   STATUS             RESTARTS   AGE
esp-operator-55f8994c76-659ch                        1/1     Running            0          27d
espmeter-deployment-6c47467cb6-5brx6                 1/1     Running            0          27d
espmeteragent-deployment-5f95f849fc-74bwj            1/1     Running            0          27d
[cloud-user@espkube-1 operator]$
````

### mkproject

The `./bin/mkproject` script is used to convert the SAS Event Stream Processing XML project into the custom resource, which is deployed as a pod on the worker nodes. When the custom resource is applied in the Kubernetes cluster using the `kubectl apply` command, it triggers the ESP Operator pod to launch the ESP Server pod with the specified ESP XML project.

Here is an example of the usage of the `mkproject` script:

```posh
[cloud-user@espkube-1 operator]$ ./bin/mkproject
Usage: ./bin/mkproject
          -x <project XML>           -- xml version of project
          -s <service name>          -- ingress servicce name
          -o <output file>           -- file to write project CR yaml to
[cloud-user@espkube-1 operator]$
````

There are three input parameters to the script: 

1.	*-x <project XML>* - This is the SAS Event Stream Processing XML Project which runs inside the pods.
2.	*-s <service name>* - This specifies the ingress service name created by the SAS Event Stream Processing Operator to allow pods to communicate via HTTP with the outside world.
3.	*-o <output file>* - This specifies the file where the output is written, that is, the custom resource YAML file.

**NOTE**: If you are using Kubernetes version 1.16.0 then make sure before running the `dodeploy` script, the API versions are corrected in the YAML files in `/deploy` folder to present errors. You need to perform this step because some APIs are deprecated in the higher version of Kubernetes.

```sh
apiVersion: extensions/v1beta1   # used for Kubernetes version 1.14.1
apiVersion: apps/v1              # used for Kubernetes version 1.16.0
````

The following command would provide all the api-versions available for the installed Kubernetes version.

```sh
# kubectl api-versions
````

## Prerequisites

Before we deploy the ESP operator and start pods running the ESP server container on the worker nodes, we must configure a few more items.

### Set up NFS

We will set up NFS as our persistence volume shared among all the nodes in the cluster.  The volume is used by the SAS Event Stream Processing metering server and to store files that are used by the models running in the ESP server at worker nodes. During the execution of the `./dodeploy` script, it takes the configuration from the `meter.yaml` manifest and creates the folder structure under the `/mnt/data` directory. Examine the `meter.yaml` file for details.

```sh
/mnt/data/
<namespace>
    DB
    input
    output
    billing
````
We use the same directory structure for our NFS. 

Run the following commands from the master node as root user:

```sh
# yum install nfs-utils nfs-utils-lib
# chkconfig --level 35 nfs on
# sudo systemctl  restart nfs
# sudo journalctl -xf
# sudo chmod 777 /mnt/data/
````

Add the following line in `/etc/exports` file at the master node:

```posh
/mnt/data 10.104.0.0/16(rw,sync,no_root_squash)
````

On all the worker nodes, add the following line in `/etc/fstab` file:

```posh
11.111.11.11:/mnt/data /mnt/data  nfs defaults 0 0
````

### Persistent Volume (pv) and Persistent Volume Claim (pvc)

Kubernetes provides storage management using two API resources: `PersistentVolume` and `PersistentVolumeClaim`.

`PersistentVolume` is a storage space provisioned by the Kubernetes cluster administrator. They are configured using the `StorageClass` (dynamically) or by the administrator. Before you can start using them, they must be attached using `PersistentVolumeClaim`.

`PersistentVolume` is created using a filesystem and specifying a size and other characteristics, such as VolumeID and VolumeName. There are various types of `PersistentVolume` [plugins](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes). In our examples, we use NFS, which we have configured previously.

Following is the code snippet of `pv.yaml` (created in the `/deploy` folder) that can be used directly. If it is not present, you can copy the code from here. In the file, we define the name of the volume, the namespace it belongs to, the dedicated path where the NFS is mounted, and its capacity.

```sh
#
# This is the esp-pv that esp component pods make use of.
#

apiVersion: v1
kind: PersistentVolume
metadata:
   name: esp-pv-volume   # name of the pv 
   namespace: espkube    # namespace where the p vis applied
   labels:
     type: local
spec:
   storageClassName: manual
   accessModes:
     - ReadWriteMany  # esp, studio and streamviewer can all write to this space
   hostPath:
     path: "/mnt/data/"
   capacity:
     storage: 5Gi  # volume size requested
````

Now apply the `pv.yaml` file:

```sh
# kubectl apply -f deploy/pv.yaml
````

See a description of the volume using the following command

```sh
#  kubectl -n espkube describe pv
````

Now we will apply the claim for using the `PersistentVolume`. This is done using the `pvc.yaml` manifest.
This file is present in the `/deploy` directory.

```sh
#
# This is the esp-pv claim that esp component pods make use of.
#
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
   name: esp-pv
   namespace: espkube
spec:
   storageClassName: manual
   accessModes:
     - ReadWriteMany # esp, studio and streamviewer can all write to this space
   resources:
     requests:
       storage: 5Gi  # volume size requested
````

Now apply the `pvc.yaml` file:

```sh
# kubectl apply -f deploy/pvc.yaml
````

See a description of the volume using the following command:

```sh
# kubectl -n espkube describe pvc
````

We use the persistent volume that we have configured at location `/mnt/data/espkube`. It is used to store the database files, input and output files used for the ESP projects.

### Kubernetes Metrics Server

You must explicitly install and configure the Kubernetes metrics server for monitoring performance - CPU utilization (cores), memory usage (bytes) and resource consumption for [Horizontal Pods Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).

Create a folder called `metricsServer` in your project directory (or in a directory of your choice) and download the TAR file in it using the following commands:

```sh
# cd metricsServer
# wget https://github.com/kubernetes-incubator/metrics-server/archive/v0.3.4.tar.gz
# tar -zxvf v0.3.4.tar.gz
````

Before starting the metrics server pod, make the following changes (highlighted) in the file `metrics-server-deployment.yaml` in the following location: `metricsServer/metrics-server-0.3.4/deploy/1.8+/`

```posh
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server-amd64:v0.3.4
        imagePullPolicy: Always
        command:
            - /metrics-server
            - --metric-resolution=30s
            - --requestheader-allowed-names=aggregator
            - --kubelet-insecure-tls
            - --kubelet-preferred-address-types=InternalDNS,InternalIP,ExternalDNS,ExternalIP,Hostname
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
````

Now apply and start the metrics server pod:

```sh
# kubectl apply -f metrics-server-0.3.4/deploy/1.8+/
````

The metrics server runs as a pod that is deployed on the Kubernetes namespace `kube-system`.

```sh
# kubectl get deployment metrics-server -n kube-system
````

You will see the metrics server pod running along with the other pods in `kube-system`.

```posh
[cloud-user@espkube-1 operator]$ kubectl -n kube-system get pods
NAME                                                                        READY   STATUS    RESTARTS   AGE
..
..
..
metrics-server-67554bbfdf-xvcpf                                             1/1     Running   0          27d
[cloud-user@espkube-1 operator]$
````

You can execute the following command to see the pod, the deployment, and the service running for the metrics server. 

```posh
[cloud-user@espkube-1 operator]$ kubectl get pods,svc,deployments -n kube-system | grep metrics-server

pod/metrics-server-67554bbfdf-xvcpf         1/1       Running            0                     27d
service/metrics-server   ClusterIP   10.103.215.110    <none>         443/TCP                  27d
deployment.extensions/metrics-server        1/1           1              1                     27d
[cloud-user@espkube-1 operator]$
````

Get cluster information using the following command:

```posh
[cloud-user@espkube-1 operator]$ kubectl cluster-info
Kubernetes master is running at https://11.111.11.11:6443
KubeDNS is running at https://11.111.11.11:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://11.111.11.11:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
[cloud-user@espkube-1 operator]$
````

Note that it shows KubeDNS instead of CoreDNS. This is a known bug in Kubernetes. 

## The Ingress
In this section, we will discuss the integration of ingress resources and ingress controller in our Kubernetes cluster to expose the applications running in the pods to the outside world and enable the users to access them over HTTP/S.

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) means exposing the HTTP and HTTPS routes from outside the Kubernetes cluster to the applications running inside the cluster.

### The Ingress Resource
The ingress resources are used to define the rules for traffic routing. In other words, we configure the ingress resources with definitions to direct the incoming traffic to the right applications running in the Kubernetes cluster. 

An Ingress resource is completely independent of the application. This means you can create, deploy and delete the resource separately of the application. This enables the users to consolidate all the ingress routing rules in one place. 

Ingress Resource is an L7 load balancer that routes the specific client request traffics to the targeted application in the cluster. This allows exposing several applications on the same IP address to the outside world. 

The ingress resource supports the following features:

1.	*Content-based Routing*: it supports host-based routing (i.e., App1.domain.com, App2.domain.com) and path-based routing (i.e., /service1, /service2)
2.	*TLS/SSL termination*: support for each hostname (i.e., App1.domain.com)

The ingress resource by itself has no effect. You need an ingress controller to execute the routing definitions defined in the ingress resource. 

The figure below presents the ingress.

![Ingress](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/archImages/ingress.png)

### The Ingress Controller
The [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) is an application that runs in a cluster and is responsible for managing the ingress resources. Most of the cloud providers such as AWS, Azure, and GCP provide their own implementation for the ingress controllers. There are many other implementations out there that can be used in any cloud environment. However, the most popular one is the Nginx Ingress Controller which we will discuss next. 

### The Nginx Ingress Controller
The [Nginx ingress controller](https://github.com/nginxinc/kubernetes-ingress) creates a bunch of nginx pods. When an ingress is created, the controller translates the provided ingress definitions in the ingress resource as nginx configuration parameters and applies them to the created nginx pods. This facilitates directing the incoming request traffic to the right application pod. 

Nginx ingress controller supports the standard ingress features such as content-based routing and TLS/SSL termination. In addition to HTTP, it also supports Websockets, gRPC, TCP and UDP applications. 

The following section will provide the commands and step to configure ingress resources and nginx ingress controller. 

### Configuration and Integration of the Ingress
To set up the nginx ingress controller we have to run the manifests provided by [Nginx](https://docs.nginx.com/nginx-ingress-controller/overview/). There are primarily a few manifests that deploy the basic default configuration. Users have two options to deploy the ingress controller, i.e., [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) and [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

In our configuration, we will use Deployment and configure a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) service for accessing the ingress controller pods. 
We have used the files from the [repository](https://github.com/nginxinc/kubernetes-ingress) that are maintained by Nginx. These are ready to use files for a simple setup. 

You can also follow the steps provided at the nginx ingress controller webpage for [installation with manifests](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/).

Clone the nginx ingress controller repository and go to the `deployments` folder:

```sh
# git clone https://github.com/nginxinc/kubernetes-ingress/
# cd kubernetes-ingress/deployments
# git checkout v1.7.0
````

Using the next few manifests, we will create a namespace for the ingress controller, a service account, cluster role and bindings for the service account. We also configure the default server which serves all the requests for domains which do not have match any of the defined ingress rules. It handles all the incoming URL paths/ports/hosts that are not configured by an ingress resource. And finally, we create a config map for customizing nginx configuration.

```sh
# kubectl apply -f common/ns-and-sa.yaml
# kubectl apply -f rbac/rbac.yaml
# kubectl apply -f common/default-server-secret.yaml
# kubectl apply -f common/nginx-config.yaml
````

Let’s now run the ingress controller using Deployment.

```sh
# kubectl apply -f deployment/nginx-ingress.yaml
````

The manifests will create the namespace `nginx-ingress`. To confirm if everything is running fine, run the command below:

```posh
[cloud-user@espkube-1 ~]$ kubectl get all -n nginx-ingress
NAME                            READY   STATUS    RESTARTS   AGE
nginx-ingress-f667d766f-xrv87   1/1     Running   0          34s
[cloud-user@espkube-1 ~]$
````

Here you can notice, that there is only one nginx ingress controller. By default, the deployment manifest `deployment/nginx-ingress.yaml` creates only one ingress controller pod. If you wish to have more, change the number of the replicas in the manifest.

You can verify the nodes on which the nginx ingress controller is running:

```sh
# kubectl -n nginx-ingress get pod -o wide
````

The output will be as follows:
```posh
[cloud-user@espkube-1 ~]$ kubectl -n nginx-ingress get pod -o wide
NAME                            READY   STATUS    RESTARTS   AGE      IP            NODE        NOMINATED NODE          READINESS   GATES
nginx-ingress-f667d766f-xrv87   1/1     Running   0          19h   10.244.2.165  espkube-3.sas.com   <none>                 <none> 
[cloud-user@espkube-1 ~]$
````

Let’s now configure the NodePort service to get access to the ingress controller pods.
```sh
# kubectl create -f service/nodeport.yaml
````

Kubernetes will randomly allocate two ports from the range `[30000-32767]` for all the worker nodes in the cluster. Ports 80 and 443 of ingress controller for HTTP and HTTPs, respectively, are mapped to the randomly allocated ports by the NodePort to access the applications running in the worker nodes.

We will now set up the [externalIPs](https://kubernetes.io/docs/concepts/services-networking/service/#external-ips) in the service specs of manifest `service/nodeport.yaml`. This will make nginx become available on both the service port and NodePort. We will add the IP address of all the nodes in the cluster under `spec:` (Make sure of the indentation)

```sh
  externalIPs:
  - 11.111.11.11
  - 22.222.22.22
  - 33.333.33.33
````
Run the following command to verify the ingress setup:
```posh
[cloud-user@espkube-1 ~]$ kubectl get all --namespace=nginx-ingress
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-f667d766f-xrv87   1/1     Running   0          24h

NAME                      TYPE       CLUSTER-IP     EXTERNAL-IP                                PORT(S)                      AGE
service/nginx-ingress   NodePort   10.96.229.29   11.111.11.11,22.222.22.22,33.333.33.33   80:32171/TCP,443:30335/TCP        24h

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-ingress   1/1     1            1           24h

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-ingress-f667d766f   1         1         1       24h 
[cloud-user@espkube-1 ~]$
````

In the service, you can observe the set ports (32171 and 30335) and the externalIPs.

In case of any problem, use the commands from the [nginx ingress controller troubleshooting](https://github.com/kubernetes/ingress-nginx/blob/master/docs/troubleshooting.md).


## The Filebrowser

The [Filebrowser](https://filebrowser.xyz/) is a standalone or middleware application that provides file management interface within a defined directory structure. It allows access to multiple users where each user can have its directory structure. It can be used to upload, delete, preview, rename and edit the files and folders. With this tool, you can access the configured persistent store also used by the Kubernetes pods in the cluster.  You can download the [filebrowser from Github](https://github.com/filebrowser/filebrowser) and refer to its home page for more details. 

The filebrowser allows the user to perform the following tasks list below:

1.	Upload, edit, and preview the ESP project (XML) files directly to the persistent store.
2.	Copy the input files (Binary, JSON, XML, CSV and others) into the persistent store.
3.	View and rename the output files generated by the running ESP projects.
4.	Copy large binary models (ASTORE files) for analytics to the persistent store which are accessed the ESP projects.
5.	Backup the database files written by the ESP metering server.

The filebrowser provides all the features of a file system. Now we will execute the following script which will download and install the filebrowser:

```sh
# curl -fsSL https://filebrowser.xyz/get.sh | bash
````

Configure the filebrowser to access the persistent storage which is already set up at the location `/mnt/data/espkube`:

```sh
# filebrowser -r /mnt/data/espkube/
````

After configuring the filebrowser, you can deploy the manifest to start a filebrowser pod with the file `fileb.yaml` at the location `operator/deploy`.

Execute the following command to start the filebrowser pod:

```sh
# kubectl -n espkube apply -f deploy/fileb.yaml
````

Once applied you will see the running pod as follows:

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube get pods
NAME                                  READY    STATUS      RESTARTS    AGE
espfb-deployment-56ffc898df-7pd7c     1/1      Running        0        5m9s
..
.. 
[cloud-user@espkube-1 operator]$
````

Now we will create the ingress for the filebrowser. This will allow the users to access the filebrowser pod from outside using the designated URL. The ingress file `ingress-fileb.yaml` is located at `operator/deploy`.

Run the command below to create the ingress `espfb.<namespace>.sas.com` where the `namespace` is `espkube`:

```sh
# kubectl -n espkube apply -f deploy/ingress-fileb.yaml
````

The filebrowser is fixed with the root of the filesystem to the `<namespace>`. This prevents the deployed filebrowser to access the data on the persistent storage that might belong to another namespace with a similar deployment using the same persistent storage.

You can see the ingress for the filebrowser as follows:

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube get ingress
NAME            HOSTS                 ADDRESS        PORTS   AGE
espfb    espfb.espkube.sas.com      10.97.163.71      80      27d
..
.. 
[cloud-user@espkube-1 operator]$
````

Once the ingress is up and running, the users can access the filebrowser using the URL `espfb.espkube.sas.com`. You can use `admin/admin` as the credentials for `username/password`.



