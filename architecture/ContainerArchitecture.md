# Introduction to Container Architecture

Before we dive in to Kubernetes, here is a brief introduction to container technologies, which enable Kubernetes to provide isolation of running applications. 

By definition, a container is a single isolated process running in the host’s operating system, consuming only the resources required by the applications. Containers provide similar resources and isolation benefits to those of virtual machines but are lightweight because they virtualize the host OS instead of the hardware. A container is a package of application or applications with all the dependencies, binaries, libraries and configuration files required for execution. Containers perform all the system calls on the same kernel running in the host OS. For a visual depiction, see the figure below. 

![Container Architecture](archImages/Container_Architecture.png)

Due to the low overhead of containers, they are a much better choice than VMs when running a great number of processes in isolation on the same machine. A process that runs in a container starts up immediately, within seconds, and nothing needs to be booted up.  Containers are portable and efficient, with less space consumption, using very few system resources. 

Two mechanisms provide container isolation:

**1.	Namespaces**

The use of namespaces to isolate processes ensures that each process sees only its own personal view of the system, which includes files, processes, network interfaces, hostname, and IP addresses. 

By default, each Linux system has only one namespace but allows the creation of additional namespaces and organizes resources across them. A process runs in a single namespace, only using the resources that are inside that namespace. 


**2.	Control Groups**

They monitor and limit the amount of resource consumption for each process, such as its CPU utilization, memory usage, and network bandwidth. With cgroups, a process cannot use more resources than it is configured to use, thus preventing one process from starving other processes of resources.    


