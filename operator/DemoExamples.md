# Demo Examples

Everything is set, up and running. Let’s investigate and run some demo examples now.

* [Using the ESP Operator](DemoExamples.md#using-the-esp-operator)
    * [Read/Write from/to the configured Persistent Volume](DemoExamples.md#read/write-from/to-the-configured-persistent-volume)
    * [Reading from a multi partitioned kafka topic with horizontal pods autoscaling based on resource consumption settings](DemoExamples.md#reading-from-a-multi-partitioned-kafka-topic-with-horizontal-pods-autoscaling-based-on-resource-consumption-settings)


# Using the ESP Operator

## Read/Write from/to the configured Persistent Volume

In this example, we will launch a pod with ESP Server container that runs the specified ESP XML project. The XML project reads a file using the [file connector](http://documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=espca&docsetTarget=n117h5f4vqvnk0n11rthjpnzrrcz.htm&locale=en) from the location set for the Persistent Volume. The project performs some computation on the read data and then writes the output in the same location of Persistent Volume.

In this example, you will need the following files:

*  Unzip the input file at location `operator/deploy/examples/input/array_input01.csv.gz` and copy the expanded file to the persistent volume location `/mnt/data/espkube/input`

*  The ESP XML project file `example-1.xml`

Now, we convert the XML project into the custom resource. ESP Operator gets triggered which starts the ESP server pod running the `example-1.xml`.

To create the custom resource `.yaml` file, execute the following command:

```sh
# ./bin/mkproject -x deploy/examples/example-1.xml -s array -o cr_array.yaml
````

Now deploy the created resource in a pod in the namespace `espkube` as follows:

```sh
# kubectl -n espkube apply -f cr_array.yaml
````

Execution of the apply command will show you:

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube apply -f cr_array.yaml
espserver.iot.sas.com/array created
[cloud-user@espkube-1 operator]$
````

Once applied, we will see the array pod running:

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube get pods
NAME                       READY    STATUS       RESTARTS   AGE
array-d9857479c-p9ds8      1/1      Running        0        5m9s
..
.. 
[cloud-user@espkube-1 operator]$
````

You will also see that the output file `array_output01.csv` is created at the `Persistent Volume` location `/mnt/data/espkube/output`

## Reading from a multi partitioned Kafka Topic with horizontal pods autoscaling based on resource consumption settings

In this example, we use a Kafka cluster running 3 Kafka instances and 3 zookeeper instances cohabiting with the Kubernetes cluster. 
We use [Strimzi Kafka Operator](https://github.com/strimzi/strimzi-kafka-operator) to run Kafka in Kubernetes Pods. 

The example demonstrates the Horizontal Pod Autoscaling for a stateless application running in the ESP Server Pods. 

Lets now set the Kafka Cluster. We create a folder named `kafka` in your project directory (or a directory of your choice) to install Kafka Operator

```sh
# cd kafka
# wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.14.0/strimzi-0.14.0.tar.gz
# tar -zxvf strimzi-0.14.0.tar.gz
````

Now we need to perform role binding which gives Strimzi a service account with required access rights. And thereafter, apply the Kafka cluster operator.

```sh
# cd strimzi-0.14.0
# sed -i 's/namespace: .*/namespace: espkube/' install/cluster-operator/*RoleBinding*.yaml
# kubectl -n espkube apply -f install/cluster-operator
````

In this example, we now create the 3 broker nodes Kafka Cluster with 3 Zookeeper and create a multi-partitioned Kafka topic. We create 10 partitions in the topic. 

The data from the topic will be used by the ESP Server pod that we will see soon later in this example.

Now create a manifest `create-cluster.yaml` and copy the code below.

```posh
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: kafka-cluster
spec:
  kafka:
    version: 2.2.1
    replicas: 3
    listeners:
      plain: {}
      tls: {}
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      log.message.format.version: '2.2'
    storage:
      type: ephemeral
  zookeeper:
    replicas: 3
    storage:
      type: ephemeral
  entityOperator:
    topicOperator: {}
    userOperator: {}
````

Apply the file now in the same namespace `espkube`

```sh
# kubectl -n espkube apply -f create-cluster.yaml
````

Now create `create-topic.yaml`. This creates a Kafka topic called `demoinputdata` with 10 partitions with the replication factor of 1. That means only one copy of the topic.

```posh
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  name: demoinputdata
  labels:
    strimzi.io/cluster: kafka-cluster
spec:
  partitions: 10
  replicas: 1
  config:
    retention.ms: 600000
    segment.bytes: 1073741824
````

Apply the file now in the same namespace `espkube`

```sh
# kubectl -n espkube apply -f create-topic.yaml
````

Now when you will run the execute the following command, you will see the 3 kafka-cluster pods, 3 zookeeper pods, strimzi-cluster-operator and kafka-cluster-entity-operator along will all the previous started pods.

```posh
[cloud-user@espkube-1 operator]$ kubectl get pods -n espkube
NAME                                                 READY   STATUS             RESTARTS   AGE
..
..
kafka-cluster-entity-operator-b5f64b578-9ws9g        3/3     Running            0          27d
kafka-cluster-kafka-0                                2/2     Running            0          27d
kafka-cluster-kafka-1                                2/2     Running            0          27d
kafka-cluster-kafka-2                                2/2     Running            0          27d
kafka-cluster-zookeeper-0                            2/2     Running            0          27d
kafka-cluster-zookeeper-1                            2/2     Running            0          27d
kafka-cluster-zookeeper-2                            2/2     Running            0          27d
strimzi-cluster-operator-6b5874876-zvw46             1/1     Running            10         27d
[cloud-user@espkube-1 operator]$ 
````

The figure below is an illustration of this example. 

![Kafka Example](archImages/Kafka_Example-_HL_Architecture_with_ESP_6.2_Kubernetes.png)

In this example, we need to launch two different ESP XML projects. 

### Launch the first XML Project

The first XML project reads from all the 10 partitions of the Kafka topic `demoinputdata` in a round-robin fashion. The project uses [Kafka Connector](http://documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=espca&docsetTarget=p0sbfix2ql9xpln1l1x4t9017aql.htm&locale=en) to read the data from the topic. It then performs some computation of the read values.

Create the custom resource for the XML project `example-2.xml` present `operator/deploy/examples` folder:

```sh
# ./bin/mkproject -x deploy/examples/example-2.xml -s source -o cr_source.yaml
````

Before applying the custom resource file, make sure the autoscaling parameters in the `cr_source.yaml` file conforms to the underlying resources, such as CPU and memory. 

We purpose to do so is allow the source pods to scale automatically to the defined number of replicas when the currently running pod cannot handle all the incoming workload and exceeds the utilization of CPU and memory defined in the custom resource.

We define the number of replicas equals to 10, the same as the number of partitions in the topic.

```posh
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
    autoscale:
      minReplicas: 1
      maxReplicas: 10
      metrics:
      - type: Resource
        resource:
          name: cpu
          target:
            type: Utilization
            averageUtilization: 50
````

**NOTE**: Make sure of the parameters set here.

Now apply the custom resource and start the pod.

```sh
# kubectl apply -f cr_source.yaml
````

Once applied, you will see the source pod running with other pods. 

```posh
[cloud-user@espkube-1 operator]$ kubectl get pods -n espkube
NAME                                                 READY   STATUS             RESTARTS   AGE
..
source-7b86cd9d84-rft4w                              1/1     Running            0          26s
..
[cloud-user@espkube-1 operator]$
````

Use the following command to see the resource consumption by the source pod:

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube top pods
NAME                                            CPU(cores)   MEMORY(bytes)
..
source-7b86cd9d84-rft4w                         21m          88Mi
..
[cloud-user@espkube-1 operator]$
````

You can notice the consumption of CPU cores and memory by the source pod. Notice that the consumptions are low at this point. It uses only 21 milli-cpus. It is because the Kafka topic being read by the ESP Server is empty and there is no data to process.

### Launching the second XML project

The second XML file reads a `.csv` file and loads the data into the Kafka topic `demoinputdata` using the Kafka Connector. 

In this example, you will need the following files:


*  Unzip the input file at location `operator/deploy/examples/input/kafka_input01.csv.gz` and copy the expanded file to the persistent volume location `/mnt/data/espkube/input`

*  The ESP XML project file `example-3.xml`

Create the custom resource for the XML project `example-3.xml` present `/operator/deploy/examples` folder:

```sh
# ./bin/mkproject -x deploy/examples/example-3.xml -s sink -o cr_sink.yaml
````

Now apply the custom resource and start the pod which will start flooding the configured Kafka.

```sh
# kubectl apply -f cr_sink.yaml
````

After a short while, we will see both sink and source running. And note that CPU is now consuming 352 milli-cpus:

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube top pods
NAME                                            CPU(cores)   MEMORY(bytes)
..
sink-77777897cf-tz9dd                           502m         33Mi
source-7b86cd9d84-rft4w                         352m         31Mi
..
[cloud-user@espkube-1 operator]$
````

We have configured source custom resource to scale up to 10 pods and in some time, you see all the 10 source pods running:

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube top pods
NAME                                            CPU(cores)   MEMORY(bytes)
..
sink-77777897cf-tz9dd                           498m         41Mi
source-859dd6c4b6-2sr4w                         125m         32Mi
source-859dd6c4b6-55mp8                         130m         33Mi
source-859dd6c4b6-8jfwc                         125m         29Mi
source-7b86cd9d84-rft4w                         130m         30Mi
source-859dd6c4b6-c2bp6                         128m         108Mi
source-859dd6c4b6-gz8v9                         135m         30Mi
source-859dd6c4b6-ksjd2                         133m         30Mi
source-859dd6c4b6-m2c4x                         198m         38Mi
source-859dd6c4b6-p24mm                         135m         29Mi
source-859dd6c4b6-qm82r                         127m         29Mi
..
[cloud-user@espkube-1 operator]$
````

You will notice that number source pods REPLICAS increase to 10 pods and processing the incoming data from the Kafka topic.

Run the following command to verify the max replicas:

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube get hpa
NAME     REFERENCE         TARGETS    MINPODS MAXPODS   REPLICAS      AGE
sink     Deployment/sink   197%/50%     1         1         1        5m11s
source   Deployment/source   54%/50%    1         10        10         7m
[cloud-user@espkube-1 operator]$
````

Now, when you will delete the sink pod, you will see the source pods will drop 1.

Delete sink with the following command:

```sh
# kubectl delete -f cr_sink.yaml
````

Verify using `hpa` command as follows. 

```posh
[cloud-user@espkube-1 operator]$ kubectl -n espkube get hpa
NAME     REFERENCE           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
array    Deployment/array    0%/50%    1         1         1          25m
source   Deployment/source   9%/50%    1         10        1          10m
[cloud-user@espkube-1 operator]$
````

The number of REPLICAS is now 1 for source pods. And sink pod is no more present.
