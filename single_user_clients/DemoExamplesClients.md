In this section, we will use the web clients SAS Event Stream Processing Studio and SAS Event Stream Manager to create, deploy, test, and manage the ESP projects.

* [Working with Web Clients](DemoExamplesClients.md#working-with-web-clients)
    * [SAS Event Stream Processing Studio](DemoExamplesClients.md#sas-event-stream-processing-studio)
    * [SAS Event Stream Manager](DemoExamplesClients.md#sas-event-stream-manager)

# Working with Web Clients

## SAS Event Stream Processing Studio
Open the SAS ESP Studio in the web browser (Google Chrome preferred) using the url http://espstudio.espkube.sas.com/SASEventStreamProcessingStudio/

* The three cloud native deployment [examples](operator/DemoExamples.md) we deployed previously can now be seen under the `ESP Servers` tab in the Studio. In the server properties, you can notice the `host` as `array.espkube.sas.com` which is nothing but the ingress for this deployment.

![Cloud Native Deployments in ESP Servers](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/single_user_clients/images/ESPServersStudio.png)

* In the ESP studio, you can create your own model using the windows in the left panel along with the editable XML in the right panel and then deploy it in the Kubernetes namespace `espkube`.

Note that this ESP Model is not provided. Users can create their own models.

![Create and Deploy the Model](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/single_user_clients/images/ModelInStudio.png)

* Let's test the model now. Once you enter the test mode, you can click the `Run Test` on top right which will allow you to edit the deployment configurations for the model. Here you can configure the memory and CPU recources request and limits, and autoscaling parameters. 

![Configure the Deployment Settings](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/single_user_clients/images/deploymentSettings.png)

* As soon the model starts running, you can see a new server pops up in the `ESP Servers` tab. In our case, it is `Lookup_and_Aggregation_Model`.

![ESP Server](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/single_user_clients/images/lookupAggServer.png)

A new pod is started in our namespace. Verify the same with the command below:

```posh
[cloud-user@espkube-1 operator]$ kubectl get pods -n espkube
NAME                                                 READY   STATUS    RESTARTS   AGE
..
..
..
xa76129070ec550a0c9ac36560605fab3-7c48fd4f45-tdkr8   1/1     Running   0          19s
````

Note the host address for the server. It is same as the ingress hostname in kubernetes environment.

```posh
[cloud-user@espkube-1 operator]$ kubectl get ingress -n espkube
NAME                                HOSTS                                               ADDRESS        PORTS   AGE
..
..
..
xa76129070ec550a0c9ac36560605fab3   xa76129070ec550a0c9ac36560605fab3.espkube.sas.com                  80      22s
````

When you stop the project, the server is also deleted from the Kubernetes cluster.

## SAS Event Stream Manager

Open the SAS Event Stream Manager in the browser using the url
http://esm.espkube.sas.com/SASEventStreamManager

* We will now create Deployment under the `Deployments` tab. As a sample example, we have created a deployement called `testDeployment`. Note that there is no running project currently.

![Create testDeployment](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/single_user_clients/images/testDeployment.png)

* In the `Projects` tab we will add a project. We will download the project we created in ESP Studio and upload it here. We have to do this because the ESP Studio runs in Standalone mode and Event Stream Manager cannot detect the projects automatically.

![Load the ESP Project](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/single_user_clients/images/loadProject.png)

* Now go to the Deployment `testDeployment` and click on the cloud icon shown in the green box. Select option `Load and Start Project in Cluster`.

![Load the Project in Deployment](https://github.com/sassoftware/iot-esp-kubernetes-reference-architecture-guide/blob/master/single_user_clients/images/startProject.png)

