# Kubernetes Cluster Environment Setup

We will now provide system requirements and step-by-step instructions to install Kubernetes and Docker and change some settings. We will also walk through some examples.

* [Hardware Requirements](KubernetesClusterEnvironmentSetup.md#hardware-requirements)
* [Software Requirements](KubernetesClusterEnvironmentSetup.md#software-requirements)
* [Configuration of Cluster with Kubernetes and Docker](KubernetesClusterEnvironmentSetup.md#configuration-of-cluster-with-kubernetes-and-docker)
    * [Disable SELinux and set rules for firewall](KubernetesClusterEnvironmentSetup.md#disable-selinux-and-set-rules-for-firewall)
    * [Add Nodes Information](KubernetesClusterEnvironmentSetup.md#add-nodes-information)
    * [Disable Swap Memory](KubernetesClusterEnvironmentSetup.md#disable-swap-memory)
    * [Install, Start and Enable Docker](KubernetesClusterEnvironmentSetup.md#install-start-and-enable-docker)
    * [Install Kubernetes](KubernetesClusterEnvironmentSetup.md#install-kubernetes)
    * [Start and Enable Kubernetes Nodes](KubernetesClusterEnvironmentSetup.md#start-and-enable-kubernetes-nodes)
    * [Updating Kubernetes Configuration on Master Node](KubernetesClusterEnvironmentSetup.md#updating-kubernetes-configuration-on-master-node)
    * [Configuring the Master Node](KubernetesClusterEnvironmentSetup.md#configuring-the-master-node)
    * [Deploy a Pod CNI Network using Flannel](KubernetesClusterEnvironmentSetup.md#deploy-a-pod-cni-network-using-flannel)
    * [CoreDNS Settings](KubernetesClusterEnvironmentSetup.md#coredns-settings)
    * [Configuring the Worker Nodes](KubernetesClusterEnvironmentSetup.md#configuring-the-worker-nodes)
    * [Verifying the cluster](KubernetesClusterEnvironmentSetup.md#verifying-the-cluster)

## Hardware Requirements

1.	An active cluster on a cloud platform (AWS EC2, Azure, GCP, OpenStack etc.) or on-prem VMs. [SAS employees can request an environment from here](http://sww.sas.com/sites/it/cloud-services/).
2.	Three servers running Red Hat Enterprise Linux 7.x or CentOS 7.x. 
3.	The minimum configuration of each server to run and test the examples - 4 vCPUs, 4 GB RAM and 10 GB disk space. At least 2 cores are required on each node to install Kubernetes.
4.	Root privileges to all the servers.

All three servers will form a cluster with one master node and two worker nodes. All these nodes will have Kubernetes and Docker installed. The following steps will help you prepare a working environment to run some SAS Event Stream Processing projects in multiple pods. 

## Software Requirements

1. Kubernetes version 1.14.1 or later
2. Docker version 19.03.2 or later
3. [Get ESP-Kubernetes Project from the github repository](https://github.com/sassoftware/esp-kubernetes)

## Configuration of Cluster with Kubernetes and Docker

In this section we prepare the environment to install Kubernetes and Docker and create a three nodes cluster. 

### Disable SELinux and set rules for firewall

Log in as root on your Kubernetes master node and disable SELinux: 

```sh
# setenforce 0
# sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

Configure firewall settings as follows:

```sh
# yum install firewall-cmd
# yum whatprovides firewall-cmd
# firewall-cmd --permanent --add-port=6443/tcp
# firewall-cmd --permanent --add-port=2379-2380/tcp
# firewall-cmd --permanent --add-port=10250/tcp
# firewall-cmd --permanent --add-port=10251/tcp
# firewall-cmd --permanent --add-port=10252/tcp
# firewall-cmd --permanent --add-port=10255/tcp
# firewall-cmd --reload
# modprobe br_netfilter
# echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
# iptables -L
# sysctl --system
```

### Add nodes information 

In case you do not have your own DNS server, add the IP addresses and names of the master and worker nodes in the `/etc/hosts` file. The names are used for referencing the nodes in the cluster.

````
11.111.11.11 espkube-1.sas.com espkube-1
22.222.22.22 espkube-2.sas.com espkube-2
33.333.33.33 espkube-3.sas.com espkube-3
````

### Disable Swap Memory

You must disable the swap memory at the master node.  Kubernetes does not perform well on a system that uses swap memory. Execute the following command to disable the swap memory:

```sh
# swapoff -a 
```

To disable the swap permanently on the master node, edit the `/etc/fstab` file. Comment the following line, if present:

```sh
#/dev/mapper/hakase--labs--vg-swap_1 none      swap    sw       0    0 
```

### Install, Start and Enable Docker

To install `docker` using `yum`, execute the following commands from the master node and both the worker nodes as `sudo`. 

```sh
# yum install -y yum-utils device-mapper-persistent-data lvm2
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum --showduplicates list docker-ce
# yum install -y docker-ce-19.03.2-3.el7
```
Verify the version of Docker with the command shown below. It should be 19.03.2 or later:

```sh
# docker version
```

Start and enable Docker on all the three nodes with the following command:

```sh
# systemctl start docker && systemctl enable docker
```

### Install Kubernetes

Kubernetes packages are not always available by default on CentOS 7 and Red Hat Enterprise Linux 7 repositories. Execute the following command from all three nodes in the cluster to configure the package:

```sh
# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
> enabled=1
> gpgcheck=1
> repo_gpgcheck=1
> gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
> https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
> EOF
```
Install the following three Kubernetes components:

1.	`kubelet`, a service that runs on all the nodes. 

2.	`kubeadm`, bootstraps a minimum viable Kubernetes cluster conforming to best practices. It is a tool to install and configure various components in a standard way and test applications. 

3.	`kubectl` is a command-line interface to run commands against the Kubernetes cluster through the API Server. It also controls the cluster manager. 

Run the command below:

```sh
# yum --showduplicates list kubectl
# yum install -y kubelet-1.14.1-0 kubeadm-1.14.1-0 kubectl-1.14.1-0 --disableexcludes=kubernetes --nogpgcheck
```

### Start and Enable Kubernetes nodes

On all the servers, start and enable the Kubernetes nodes:

```sh
# systemctl start kubelet && systemctl enable kubelet
```

To verify the Kubernetes configuration, run the following command:

```sh
# journalctl -xf
```

### Updating Kubernetes Configuration on Master Node

Now, we will add an environment variable `cgroup-driver=systemd/cgroup-driver=cgroupfs` in the Kubernetes configuration file. The file is stored at the following location on the master node: `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`.

Execute the following command only on the master node:

```sh
# sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

### Configuring the Master Node

Now that we have completed the installation of required packages on all the nodes, it is time to start the configuration. 

#### *Step 1: Initialization*

Execute `kubeadm init` which does the following:

1.	Applies labels and taints to the configuration of the master node so that no workloads are assigned to it.
2.	Generates the token that worker nodes can use to join the master node in processing incoming workloads.
3.	Performs all the necessary configurations for the worker nodes joining the master node using either [bootstrap tokens](https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/) or [TLS bootstrap](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/) mechanism.

Run the following command to initialize the Kubernetes cluster services. `kubeadm` will automatically pull all the necessary initialization images.

```sh
# kubeadm init --ignore-preflight-errors=SystemVerification --apiserver-advertise-address=11.111.11.11 --pod-network-cidr=10.244.0.0/16 --token-ttl 0
```

In the above `init` command, it is worth noting a few settings: 

1.	`--ignore-preflight-errors` runs a series of pre-flight checks to validate the correctness of the system. Some checks trigger warnings while others can result in errors forcing `kubeadm` to exit the initialization until the mentioned problems are resolved.  

We have set this check to `SystemVerification` which allows Kubernetes to launch an unsupported Docker version and initiate error-free. 

2.	`--apiserver-advertise-address` sets the IP address Kubernetes should advertise its API Server on. We set it as the IP address of the master node. If this is not set, then the default network interface will be used.

3.	`--pod-network-cidr` specifies the virtual network range of IP addresses that the pod network must use. We set this parameter to `10.244.0.0/16` as we are using “Flannel” Container Network Interface (CNI) network which we will discuss shortly. Flannel uses the above subnet by default.

4.	We use bootstrap tokens to establish a secure bidirectional trust between the worker nodes and the master node. By default, `ttl` for any generated token is 24hours. However, this can be overridden by setting the time during the initialization. We have set `--token-ttl` to `0` which means that the token never expires.

You will get the output shown below on executing the `kubeadm init` command:

```posh
…….
…….
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 11.111.11.11:6443 --token r1mpah.53wp03oyg1iofu9b --discovery-token-ca-cert-hash sha256:352f5ff2a9cff2cb561b2ea9e9fd2002fb4f2948265078aafe93ef017e2e45be

[root@espkube-1 cloud-user]#
```

#### *Step 2: Run the commands from the output of `kubeadm init`*

According to the output, we can run the commands on the master node as a regular user.

```sh
# mkdir -p $HOME/.kube
# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# chown $(id -u):$(id -g) $HOME/.kube/config
````

### Deploy a Pod CNI Network using Flannel

Kubernetes does not provide any default network implementation but defines the fundamental requirements. There are many available implementations for [pod networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/); Flannel is one of them and is the simplest. 

Flannel creates an overlay network on top of the host network. All the pods in the Kubernetes cluster are then assigned one IP address each used for pod-to-pod, pod-to-service, and external-to-service communications. 

Previously, we set `--pod-network-cidr=10.244.0.0/16` because [kube-flannel.yaml](https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml) has hardcoded it as the network value. If you want to use another subnet or the default of Kubernetes, you must modify the yaml file and apply the changes. However, it is easier to use the Flannel settings without modifying the yaml file.

Make sure that the content of `/run/flannel/subnet.env` is as follows:

```sh
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
````

![Flannel Network](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/archImages/FlannelESPKube.png)

Run the following command to setup the flannel pod networking as presented in the above figure.

```sh
# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```

or 

```sh
# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

Without any CNI Network, the master node of the cluster would never be Ready.

### CoreDNS Settings

We now set the CoreDNS. The command opens the configuration file to edit:

```sh
# kubectl edit configmap coredns -n kube-system
```

Update the file with the IP addresses of all the nodes in the Kubernetes cluster under `hosts` and save the changes once completed.

```sh
        health
        hosts {
           11.111.11.11
           22.222.22.22
           33.333.33.33
           fallthrough
        }
        ready
        …
        …
```

We selected CoreDNS over Kube-DNS for our environment. In the background, they both perform the same task, but a CoreDNS implementation has better resource consumption and overall performance. 

1.	CoreDNS has a single container per instance, while Kube-DNS has three. Kube-DNS has high memory requirements, adding performance overhead. 
2.	For caching, CoreDNS uses multi-threaded Go while Kube-DNS uses dnsmasq which is single threaded C.
3.	CoreDNS uses negative caching for handling domain name searches. Kube-DNS does not.


### Configuring the Worker Nodes

Adding a worker node to a cluster is trivial. Whenever the master node of the cluster is initialized, its output consists of the `kubeadm join` command with the token. We have seen the command when we executed `kubeadm init` in the previous section.

```sh
# kubeadm join 11.111.11.11:6443 --token r1mpah.53wp03oyg1iofu9b --discovery-token-ca-cert-hash sha256:352f5ff2a9cff2cb561b2ea9e9fd2002fb4f2948265078aafe93ef017e2e45be
```

The output of the `kubeadm join` command is as follows:

```posh
…….
…….
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster. 
[root@espkube-2 cloud-user]#
```

### Verifying the cluster

It is important to verify that all the nodes in the cluster are operating correctly. Sometimes the cluster can fail to complete the setup for various reasons, for example if a node goes down, the network fails between the master and worker nodes, or the installation was not complete. 

To check the current state of the cluster, execute the following command to see whether all the nodes are up and running:

```sh
# kubectl get nodes
```

You will see output similar to the following:

```posh
[cloud-user@espkube-1 ~]$ kubectl get nodes
NAME                                                STATUS   ROLES    AGE   VERSION
espkube-1.sas.com                                   Ready    master   27d   v1.14.1
espkube-2.sas.com                                   Ready    <none>   27d   v1.14.1
espkube-3.sas.com                                   Ready    <none>   27d   v1.14.1
[cloud-user@espkube-1 ~]$
```

espkube-1, espkube-2, and espkube-3 are all up and running. espkube-1 is the master node (ROLES is master) while others are worker nodes (no aliasing set for them; therefore, you see <none> in ROLES).

The STATUS of all the nodes is Ready, which means that all the nodes are part of the cluster and ready to handle the workload. 

The command below will display all the pods running in all the namespaces.

```sh
# kubectl get pods --all-namespaces
```

The output of the above command is as follows:

```posh
[cloud-user@espkube-1 ~]$ kubectl get pods --all-namespaces
NAMESPACE             NAME                                        READY    STATUS        RESTARTS        AGE
kube-system   coredns-584795fc57-pjg58                             1/1     Running            0          27d
kube-system   coredns-584795fc57-r8smp                             1/1     Running            0          27d
kube-system   etcd-espkube-1.sas.com                               1/1     Running            0          27d
kube-system   kube-apiserver-espkube-1.sas.com                     1/1     Running            0          27d
kube-system   kube-controller-manager-espkube-1.sas.com            1/1     Running            0          27d
kube-system   kube-flannel-ds-amd64-7jh8n                          1/1     Running            0          27d
kube-system   kube-flannel-ds-amd64-8skgk                          1/1     Running            0          27d
kube-system   kube-flannel-ds-amd64-xgz92                          1/1     Running            0          27d
kube-system   kube-proxy-jx7dl                                     1/1     Running            0          27d
kube-system   kube-proxy-m6qd6                                     1/1     Running            0          27d
kube-system   kube-proxy-spfkm                                     1/1     Running            0          27d
kube-system   kube-scheduler-espkube-1.sas.com                     1/1     Running            0          27d
[cloud-user@espkube-1 ~]$
```

You will notice running status for coredns (one for each worker node) and kube-flannel (one for each node) pods along with kube-apiserver, kube-proxy (one for each node), kube-controller-manager and kube-scheduler.
