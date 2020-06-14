# Single-User Clients

With single-user clients, users can deploy SAS Event Stream Processing graphics clients. The directory is called `single_user_clients` in the main directory `esp-kubernetes`. It contains a set of scripts and YAML files to deploy the following clients:

1.	SAS Event Stream Processing Studio
2.	SAS Event Stream Processing Streamviewer
3.	SAS Event Stream Processing Manager

These clients are deployed in the pods in the Kubernetes cluster and run alongside an ESP server. Pods of SAS ESP Studio, SAS ESP Streamviewer, SAS ESP Manager and ESP servers can cohabit in the same worker nodes.

The deployment is an open deployment which means it is without any authentication. This must be used only by a single user or a small cooperative team.

* [Prerequisites](SingleUserClients.md#prerequisites)
* [Creating a Deployment](SingleUserClients.md#creating-a-deployment)
* [Ingress](SingleUserClients.md#ingress)

## Prerequisites

There are two mandatory requirements:

1.	Before executing the scripts to deploy the single-user clients, you must have the ESP Operator deployed and running.
2.	You must use the same Kubernetes namespace as that of the ESP Operator for deploying all the scripts from the `single_user_clients` directory. 

**NOTE**: In our configurations for ESP Operator and single-user clients we have used namespace as `espkube`.

## Creating a Deployment

We will now use the scripts in the directory `esp-kubernetes/single_user_clients` to deploy the graphical clients. The following three scripts are in the `single_user_clients/bin` folder of the repository.

1.	./bin/mkdeploy
2.	./bin/dodeploy
3.	./bin/mkproject

These scripts are similar to the ones we used for ESP Operator.

To run the commands presented from this point, you must go to `single_user_clients` directory in the main `esp-kubernetes` directory.

### mkdeploy

`single_user_clients/templates` directory consists of the `YAML` files templates for each client, SAS ESP Studio, SAS ESP Streamviewer, and SAS ESP Manager. These templates cannot be used as it is. They contain the placeholder values that must be replaced with the values corresponding to the Kubernetes cluster. The `./bin/mkdeploy` script substitutes the placeholder values with real values.

Using `mkdeploy` script:

```posh
[cloud-user@espkube-1 single_user_clients]$ ./bin/mkdeploy
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
[cloud-user@espkube-1 single_user_clients]$
````

According to the usage instructions of the script for the single-user clients deployment, we provide

1.	SAS ESP Manager image
2.	SAS ESP Studio image
3.	SAS ESP Streamviewer image
4.	SAS ESP license file
5.	SAS ESP certificate

For this section, we do not use the options for the ESP operator deployment.

For non-SAS employees, all the images can be fetched from the [SAS secure repository to your private mirror registry](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=dplyesp0phy0lax&docsetTarget=p1waqvpevifcbvn1sjcr8d15j5m0.htm&locale=en).
The [license and certificate files](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=dplyesp0phy0lax&docsetTarget=p1bte5sqqf6cban0zm2qi61y7gul.htm&locale=en) are sent to you on your business email upon making a product order for SAS Event Stream Processing for Edge Computing.

For SAS Employees, all the images can be fetched from the [SAS Container Registry](http://docker.sas.com/repositories/pdt). To obtain the license files you can [make a product order here](https://makeorder.sas.com/makeorder/orderinfo) for product SAS Event Stream Processing for Edge Computing. 

Here is an example of invoking the `mkdeploy` script from the master node:

```sh
# ./bin/mkdeploy -r -n espkube 
-e docker.sas.com/pdt/sas-esmapplication:6.2.0-20191105.1572971561506 
-t docker.sas.com/pdt/sas-espstudio:6.2.0-20191105.1572971648575 
-v docker.sas.com/pdt/sas-espstreamviewer:6.2.0-20191106.1573053230970 
-d sas.com 
-l /home/cloud-user/licenses/SASViyaV0300_9C7K7Y_Linux_x86-64.txt 
-c /home/cloud-user/certificates/SAS_CA_Certificate.pem
````

**NOTE**: The `-l` option requires a TXT license file, not a JWT file.

The license file is required by the SAS ESP Manager

After launching the `mkdeploy` script, you will see a deploy directory under `single_user_clients` directory.

### dodeploy

Before performing the deployment, it makes sure that the namespace `espkube` exists. In the absence of the namespace, the script asks to create a namespace.

```posh
[cloud-user@espkube-1 single_user_clients]$ ./bin/dodeploy
Usage: ./bin/dodeploy
          -n <namespace>             -- specify K8 namespace
[cloud-user@espkube-1 single_user_clients]$
````

Here is the sample example to use `dodeploy` script.

```sh
# ./bin/dodeploy -n espkube 
````

After the deployment you would see the ESP Studio, ESP Streamviewer and ESP Manager pods running in the `espkube` namespace.

```posh
[cloud-user@espkube-1 ~]$ kubectl -n espkube get pods
NAME                                     READY     STATUS      RESTARTS   AGE
esm-deployment-5796bc9c64-42798           1/1     Running         0       27d
espstudio-deployment-dc6665fb9-vmglh      1/1     Running         0       27d
streamviewer-deployment-86f4b48bf4-w9ctd  1/1     Running         0       47d
[cloud-user@espkube-1 ~]$
````
All the client applications take a while to start. To see the logs and monitor the startup progress of the deployments, use the following command (use the name of the pods):

```sh
# kubectl -n espkube logs -f espstudio-deployment-dc6665fb9-vmglh
````

## Ingress

During the execution of `mkdeploy` script, we provided the `-d` (Ingress domain root) parameter and the namespace. These are used to create the ingress routes to the three client applications:

1.	SAS ESP Studio: `espstudio.<namespace>.<domain>`
2.	SAS ESP Streamviewer: `streamviewer.<namespace>.<domain>`
3.	SAS ESP Manager: `esm.<namespace>.<domain>`

The corresponding full URLs using the namespace as `espkube` and domain as `sas.com` along with the context names are as follows:

1.	SAS ESP Studio: `espstudio.espkube.sas.com/SASEventStreamProcessingStudio`
2.	SAS ESP Streamviewer: `streamviewer.espkube.sas.com/SASEventStreamProcessingStreamviewer`
3.	SAS ESP Manager: `esm.espkube.sas.com/SASEventStreamManager`

Verify the ingresses using the below command:

```posh
[cloud-user@espkube-1 ~]$ kubectl -n espkube get ingress
NAME                HOSTS                          ADDRESS        PORTS   AGE
esm            esm.espkube.sas.com               10.97.163.71      80     47d
espstudio      espstudio.espkube.sas.com         10.97.163.71      80     47d
streamviewer   streamviewer.espkube.sas.com      10.97.163.71      80     47d
[cloud-user@espkube-1 ~]$
````

We can also verify accessing one of these applications using the HTTP port we have set previously.

```posh
[cloud-user@espkube-1 ~]$ curl -D- http://espstudio.espkube.sas.com:32171/SASEventStreamProcessingStudio
HTTP/1.1 302
Server: nginx/1.17.10
Date: Wed, 06 May 2020 15:05:12 GMT
Transfer-Encoding: chunked
Connection: keep-alive
Location: http://espstudio.espkube.sas.com/SASEventStreamProcessingStudio/
[cloud-user@espkube-1 ~]$
````

You can also access it without mentioning the HTTP port:
```posh
[cloud-user@espkube-1 ~]$ curl -D- http://espstudio.espkube.sas.com/SASEventStreamProcessingStudio
HTTP/1.1 302
Server: nginx/1.17.10
Date: Wed, 06 May 2020 15:08:39 GMT
Transfer-Encoding: chunked
Connection: keep-alive
Location: http://espstudio.espkube.sas.com/SASEventStreamProcessingStudio/
[cloud-user@espkube-1 ~]$
````
It is worth noting that the `Location` specified in both are same.

You can test accessing other applications, SAS ESM and SAS ESP Streamviewer similarly.

Everything is set, up and running. Let’s investigate the example now.
