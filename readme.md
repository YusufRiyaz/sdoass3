# Name: Yusuf Riyaz
## student ID: s3615021

### Systems Deployment and Operations
### RMIT
### Assignment 3
### Task A
This application will now be deployed in [Docker](https://www.docker.com/) and hosted on [Kubernetes](https://kubernetes.io/). We will be using a Helm as the package manager to deploy the services  and application to Kubernetes. 

Let us look into what each tool is and how it will be used in deploying your application.


# What is Docker?

Docker is a Platform as a Service (PaaS) software that utilises Operating system level virtualisation that delivers software in packages called containers. These containers will be bundled with their own configuration files, libraries and source code files that is necessary to run the application. 

A docker container image that includes everything that is needed to run like code and runtime will be created.

For more information, visit [Docker](https://www.docker.com/).

# What is Kubernetes?

Kubernetes is a container orchestration system for automating application deployment and management. Kubernetes allows for:
* ### Deployment
* ### Scaling
* ### Monitoring

### In Kubernetes, a cluster is a set of node machines running containerized applications. If Kubernetes is running, a cluster is running. At minimum, a cluster contains a master node and a worker node. 

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

     To save costs, we will be using namespaces to assign the production environment and the non-production environment.


For more information, visit [Kubernetes](https://kubernetes.io/).


#  What is Helm?
Helm is a package manager for Kubernetes that allows developers and operators to more easily package, configure, and deploy applications and services onto Kubernetes clusters.

### Helm can 

* ### Configure software deployments
* ### Upgrade application
* ### List applications grouped by namespaces

To install the Helm CLI, visti [Install Helm](https://helm.sh/docs/intro/install/)


For more information, visit [Helm](https://helm.sh/docs/).
__________________________________________________________________________________





### AWS 
Amazon Web Service is a secure cloud services platform that offers computing power,storage solutions and content delivery.
We will be using AWS to run the kubernetes cluster as well as stand up the backend with S3 and RDS.

### Terraform
Terraform is an 'Infrastructure as Code' tool that allows you to create, update and version infrastructure safely and efficiently.

For more information, visit [Terraform.io](https://www.terraform.io/intro/index.html)


### Makefile
A makefile is a file containing a set of directives used by a make build automation tool to generate a target/goal. We will be using makefile to initialise terraform as well as confugre kuberentes later.


In the `Makefile`in `/infra`directory declare your 


## Terraform apply

We need to first start the services to AWS by running the environment infrastructure.

* Change directory to environment where the MakeFile is:

     `cd environment`

* Run makefile command 

    `Make up`

![Alt text](/screenshots/makeup.png?raw=true )

As you can see from the outputs, we would need these outputs to be defined in other folders to successfully deploy the application.





### To create charts for the application, run the following command on terminal in the root directory. We will clean up some directories that we will not need and create files needed for hosting on Kubernetes.

```
helm create chart-name
rmdir charts
rm -rf templates/*
touch values.yaml

```
### Define variables needed for the deployment as shown below:

* values.yaml

````yaml
replicaCount: 1
env: prod
image: 
dbhost:
dbusername: testuser
dbpassword: TestPass
dbname: servian

````
We will leave the image and the dbhost variables empty as we will be fetching the variables dyanmically from terraform and circleci

* deployment.yaml
````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: "app"
spec:
  selector:
    matchLabels:
      app: "app"
  replicas: {{ .Values.replicaCount }}
  template:
    metadata:
      labels:
        app: "app"
    spec:
      containers:
      - image: {{ .Values.image }}
        name: "app"
        env:
          - name: DB_HOSTNAME
            value: {{ .Values.dbhost }}
          - name: DB_USERNAME
            value: {{.Values.dbusername}}
          - name: DB_PASSWORD
            value: {{.Values.dbpassword}}
          - name: DB_NAME 
            value: {{.Values.dbname}}
        ports:
        - name: http
          protocol: TCP
          containerPort: 80

````

In `deployment.yaml` file, we are fetching the values defined in `values.yaml` with the `{{.Values.name}}` tag.

![Alt text](/screenshots/init.png?raw=true )




dynamoDb_lock_table_name = RMIT-locktable-hlpovo
ecr_url = 234004604603.dkr.ecr.us-east-1.amazonaws.com/app
kops_state_bucket_name = rmit-kops-state-hlpovo
state_bucket_name = rmit-tfstate-hlpovo