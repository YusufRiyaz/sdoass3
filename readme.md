## Yusuf Riyaz
### S3615021

This application will now be deployed in [Docker](https://www.docker.com/) and hosted on [Kubernetes](https://kubernetes.io/). We will be using a Helm as the package manager to deploy the services  and application to Kubernetes. 

Let us look into what each tool is and how it will be used in deploying your application.

# Docker

Docker is a Platform as a Service (PaaS) software that utilises Operating system level virtualisation that delivers software in packages called containers. These containers will be bundled with their own configuration files, libraries and source code files that is necessary to run the application. 

A docker container image that includes everything that is needed to run like code and runtime will be created.

# Kubernetes

Kubernetes is a container orchestration system for automating application deployment and management. Kubernetes allows for:
* Deployment
* Scaling
* Monitoring

In Kubernetes, a cluster is a set of node machines running containerized applications. If Kubernetes is running, a cluster is running. At minimum, a cluster contains a master node and a worker node. 

* ### Master node

     The master node is the machine the controls  kubernetes nodes. This is where all the tasks assignments originate.

 * ### Worker node or Slave node

     These machines will perform the tasks assigned to the master node.

  * ### Pod 

     A pod is smallest object in Kubernetes where a set of 1 or more containers are deployed to a single node.

  * ### Service

     Service allows a way to expose an application running on a set of pods as a network service.

  * ### Namespace
  
     A virtual cluster. Namespace allows kubernetes to manage multiple clusters within the same physical cluster.


# Helm






dynamoDb_lock_table_name = RMIT-locktable-hlpovo
ecr_url = 234004604603.dkr.ecr.us-east-1.amazonaws.com/app
kops_state_bucket_name = rmit-kops-state-hlpovo
state_bucket_name = rmit-tfstate-hlpovo