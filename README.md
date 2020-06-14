# The Complete Guide to SAS Event Stream Processing 6.2 with Kubernetes

The complete guide to SAS Event Stream Processing with Kubernetes provides all the necessary information and guidelines with commands to set up your own Kubernetes cluster and its peripherals. It helps the user to start the SAS ESP Operator, SAS ESP Studio, SAS ESP Streamviewer and SAS Event Stream Manager in the user-defined namespace.

## Table of Contents
* [Introduction to Container Architecture](architecture/ContainerArchitecture.md)
* [Kubernetes Architecture](architecture/KubernetesArchitecture.md)
* [Kubernetes Cluster Environment Setup](cluster_setup/KubernetesClusterEnvironmentSetup.md)
* [End-to-End ESP Kubernetes (ESP-Kube) Architecture](architecture/EndToEndESPKubeArchitecture.md)
* [ESP Operator](operator/ESPOperator.md)
    * [Demo Examples to use ESP Operator](operator/DemoExamples.md)
* [Single-User Clients](single_user_clients/SingleUserClients.md)
    * [Demo Examples for cloud-native deployments using web-clients](single_user_clients/DemoExamplesClients.md) 


## Overview

SAS Event Stream Processing 6.2 with Kubernetes is a framework for controlled and automatic deployment, management, and scaling of SAS Event Stream Processing Servers running in Docker containers across the Kubernetes cluster in a cloud environment. It provides high guarantees on availability, reliability and fault tolerance. The framework leverages features of Kubernetes for resource allocation, storage orchestration, self-monitoring and healing, and secret and configuration management.

The framework comprises of SAS Event Stream Processing Operator which allows starting, stopping, updating, deleting, and monitoring of the ESP Servers docker containers running in the Kubernetes cluster.

SAS Web Clients (SAS Event Stream Processing Studio, SAS Event Stream Processing Streamviewer and SAS Event Stream Processing Manager) also runs in the docker containers in the same cluster and allows seamless access to all the ESP Servers.

This paper provides a thorough deep dive into the SAS Event Stream Processing 6.2 Kubernetes Architecture and guidelines for users to set up their own ESP Operator framework with web clients in a Kubernetes cluster. 

### Installation
Follow the table of contents in order for step-by-step instructions to install, configure and use SAS ESP 6.2 Kubernetes Framework.

## Contact Information
Your comments and suggestions are valuable and most welcome. Contact the author at:

Divya Gupta (Divya.Gupta@sas.com)    
Solutions Architect, IoT Global Division

## Contributing

We welcome your contributions! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on how to submit contributions to this project. 

## License

This project is licensed under the [Apache 2.0 License](LICENSE).

## Additional Resources

* R&D Projects on [ESP Kubernetes](https://github.com/sassoftware/esp-kubernetes)
* [SAS Event Stream Processing](https://www.sas.com/en_us/software/event-stream-processing.html)
* Documentation for [SAS Event Stream Processing Kubernetes](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=espex&docsetTarget=n1nbfkss07x269n1in6yfnpju3i9.htm&locale=fr)
* Documentation for [SAS Event Stream Processing Studio](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=espstudio&docsetTarget=titlepage.htm&locale=en)
* Documentation for [SAS Event Stream Manager](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=esmuse&docsetTarget=titlepage.htm&locale=en)
* Documentation for [SAS Event Stream Processing Streamviewer](https://go.documentation.sas.com/?cdcId=espcdc&cdcVersion=6.2&docsetId=espvisualize&docsetTarget=titlepage.htm&locale=en)
* [SAS Support Communities](https://communities.sas.com/)
