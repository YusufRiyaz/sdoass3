# Name: Yusuf Riyaz
## Student ID: s3615021

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


### For more information, visit [Kubernetes](https://kubernetes.io/).




#  What is Helm?
Helm is a package manager for Kubernetes that allows developers and operators to more easily package, configure, and deploy applications and services onto Kubernetes clusters.

### Helm can 

* ### Configure software deployments
* ### Upgrade application
* ### List applications grouped by namespaces

To install the Helm CLI, visit [Install Helm](https://helm.sh/docs/intro/install/).


### For more information, visit [Helm](https://helm.sh/docs/).
__________________________________________________________________________________



# AWS 
Amazon Web Service is a secure cloud services platform that offers computing power,storage solutions and content delivery.
We will be using AWS to run the kubernetes cluster as well as stand up the backend with S3 and RDS.

# Terraform
Terraform is an 'Infrastructure as Code' tool that allows you to create, update and version infrastructure safely and efficiently.

### For more information, visit [Terraform.io](https://www.terraform.io/intro/index.html)


## Makefile
A makefile is a file containing a set of directives used by a make build automation tool to generate a target/goal. We will be using makefile to initialise terraform as well as confugre kuberentes later.




## Deploying environment infrastructure

We need to first start the services to AWS by running the environment infrastructure by initialising the `main.tf`file and applying via terraform.

* ###Change directory to environment where the MakeFile is:

     `cd environment`

* ### Run makefile command 

    `Make up`

![Alt text](/screenshots/makeup.png?raw=true )

### As you can see from the outputs, we would need these outputs to be defined in other folders to successfully deploy the application.

### We need to then create a cluster according to the terraform state saved in the S3 bucket as well as attaching a Iam policy.

### we can do this by invoking the following command:

 * ### Make sure you are in the right directory

     `cd environment`

* ### Run Makefile command:

    `make kube-up`

* Once the command is run, validate by running:

     `make kube-validate`

You will see the following when your cluster is ready:

![Alt text](/screenshots/kubevalidate.png?raw=true )

### For this application, a master node is paired with 2 working nodes.


## At this stage, you have now stood up the kubernetes environment to deploy the helm charts!

## Deploying database infrastructure

We will now go to the makefile in the infra folder and update our RDS and s3 

Change directoy to infra from root

`cd infra`

### Update the backend config to S3 and RDS URI as shown below:

```yaml
up:
	terraform apply --auto-approve -var environment=${ENV}

down:
	terraform destroy --auto-approve -var environment=${ENV}

init:
	terraform init --backend-config="key=state/${ENV}.tfstate" --backend-config="dynamodb_table=RMIT-locktable-hlpovo" --backend-config="bucket=rmit-tfstate-hlpovo"

```
After, run:

`make init`
`make up`

### Check the AWS console to check if the backend is configured.


### You have now set up the backend for the deployment.

Part B
## Creating Helm charts


### To create charts for the application, run the following command on terminal in the root directory. We will clean up some directories that we will not need and create files needed for hosting on Kubernetes.

```
helm create acme
rmdir charts
rm -rf templates/*
touch values.yaml

```
### Define variables needed for the deployment as shown below:

## values.yaml

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

## deployment.yaml
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
          containerPort: 3000

````

### In `deployment.yaml` file, we are fetching the values defined in `values.yaml` with the `{{.Values.name}}` tag.

## Service.yaml

This file is used to access the application and defining the ports.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: "app"
spec:
  ports:
    - port: 80
      targetPort: 3000
      protocol: TCP
  type: LoadBalancer
  selector:
    app: "app"

```
The helm chart files now contains the following:

* Deployment manifest
* Service Manifest

lets pass the image and the db host variables through circleci.

### Add the ECR value to your `config.yaml` as shown below:

```yaml

deploy-test:
    machine: true
    environment:
      ECR: 234004604603.dkr.ecr.us-east-1.amazonaws.com
      reponame: app
      NODE_ENV: test
      ENV:  test

```
### Define the ENV to `test` which will be our non-production environment

### In your deploy-test run in `config.yaml`:
````yaml
 

      - run:
          name: deploying backend infrastructure
          command: |
            cd artifacts/infra
            make init 
            make up
            terraform output endpoint
            terraform output endpoint > ../dbhost.txt
      - run:
          name: deploying application
          command: |
            helm upgrade app artifacts/acme-0.1.0.tgz -i -n test --wait --set image=$(cat artifacts/image.txt),dbhost=$(cat artifacts/dbhost.txt)
      - run: 
          name: migrate database 
          command: |
            kubectl exec deployment/app -n test -- node_modules/.bin/sequelize db:migrate --env production
  
````

```yaml

helm upgrade app artifacts/acme-0.1.0.tgz -i -n test --wait --set image=$(cat artifacts/image.txt),dbhost=$(cat artifacts/dbhost.txt)

```
### Append the output of the endpoint into a text file so that it can be called via a shell command when performing a `helm upgrade` command.

### The image tag can be set as shown above as it the image tag was appended in the `image.txt` as shown below.

```yaml
- run: 
          name: Build image
          command: |
            cd src
            export IMAGE_TAG=${ECR}/${reponame}:${CIRCLE_SHA1}
            echo ${IMAGE_TAG} > ../artifacts/image.txt
            docker build -t ${IMAGE_TAG} .
            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR}
            docker push ${IMAGE_TAG}

```

## Deploying test environment

Before we take a look at the config file in `config.yaml`, we need to create a namespace for the test environment in kubernetes.

### This can be done by using the kubectl CLI as shown below

````
kubectl create namespace test

````

### The circleci config are as shwown below:

`````yaml
deploy-test:
    machine: true
    environment:
      ECR: 234004604603.dkr.ecr.us-east-1.amazonaws.com
      reponame: app
      NODE_ENV: test
      ENV:  test
    steps:
      - checkout

      - attach_workspace:
          at: ./

      - setup-cd

`````
### We define the ECR here and the environment.
### We attach the workspace from the previous jobs as well as define our repository name.

### setup-cd is used to install dependencies and configure kops.

````yaml
      - run:
          name: deploying backend infrastructure
          command: |
            cd artifacts/infra
            make init 
            make up
            terraform output endpoint
            terraform output endpoint > ../dbhost.txt
      - run:
          name: deploying application
          command: |
            helm upgrade app artifacts/acme-0.1.0.tgz -i -n test --wait --set image=$(cat artifacts/image.txt),dbhost=$(cat artifacts/dbhost.txt)

````

### We invoke the upgrade command with helm to set the values in the `values.yaml`file respective to the variables.

````yaml

      - run: 
          name: migrate database 
          command: |
            kubectl exec deployment/app -n test -- node_modules/.bin/sequelize db:migrate --env production
       - run:
          name: set endpoint for e2e 
          command: |
            apt-get install jq -y
            export  ENDPOINT=http://$(kubectl get service/app -n test -o json | jq '.status.loadBalancer.ingress[0].hostname' | sed -e 's/^"//' -e 's/"$//')
            
  ````

### We seed the backend database using the db:migrate command. 


### Edit the workflow to include the `deploy-test ` run in the workflow.


````yaml
workflows:
  version: 2
  build-test-package:
    jobs:
      - build
      - integration:
          requires: 
             - build
      - e2e:
          requires:
            - integration
      - package:
          requires:
            - e2e
      - deploy-test:
          requires:
            - package
          filters:
            branches:
              only: master

````

Push up to your Git repository:
`````
git add .
git commit -m "deploy test"
git push

`````

You should see this once it successfully runs:

![Alt text](/screenshots/deploytest.png?raw=true )



Part D

## Deploying e2e test on non-production environment

As the e2e tests in `/src/.qawolf/tests` awaits browser launch for a endpoint endpoint variable and the local host, we need to define the deployed applcations's endpoint which can be found using the following command:

````
kubectl get service -n test


````
* `-n` is a shorthadn for namespace and test is our namespace.


![Alt text](/screenshots/getservice.png?raw=true )

The `EXTERNAL-IP` is your endpoint.

### In circleci declare your endpoint in the qawolf image environment variables as show below

````yaml
 e2e:
    docker:
      - image: qawolf/qawolf:v0.9.2
      - image: postgres:10.7
        environment:
          POSTGRES_PASSWORD: password
    environment:
      QAW_HEADLESS: true
      DB_USERNAME: postgres
      DB_PASSWORD: password
      DB_NAME: servian
      ENDPOINT: http://ac754c64f87c349e595db77d38885e21-1142031128.us-east-1.elb.amazonaws.com

````
### This will perform end to end testing with the endpoint provided. Make sure you specify the endpoint with the http protocol.

### workflow

There is nothing needed to be changed in the workflow.

### The following results will appear in the circleci console.

![Alt text](/screenshots/e2ejob.png?raw=true )
![Alt text](/screenshots/e2e.png?raw=true )



## Deploying a test environment

Deploying a test environment is similar to deploying but with one subtle difference, the environment should be defined as prod for produciton.


### This can be done by using the kubectl CLI as shown below

Create a namepspace for prod

````
kubectl create namespace prod
````


### The circleci config are as shwown below:

`````yaml
deploy-test:
    machine: true
    environment:
      ECR: 234004604603.dkr.ecr.us-east-1.amazonaws.com
      reponame: app
      NODE_ENV: test
      ENV:  test
    steps:
      - checkout

      - attach_workspace:
          at: ./

      - setup-cd

`````
### We define the ECR here and the environment.
### We attach the workspace from the previous jobs as well as define our repository name.

### setup-cd is used to install dependencies and configure kops.

````yaml
      - run:
          name: deploying backend infrastructure
          command: |
            cd artifacts/infra
            make init 
            make up
            terraform output endpoint
            terraform output endpoint > ../dbhost.txt
      - run:
          name: deploying application
          command: |
            helm upgrade app artifacts/acme-0.1.0.tgz -i -n prod --wait --set image=$(cat artifacts/image.txt),dbhost=$(cat artifacts/dbhost.txt)

````

### We invoke the upgrade command with helm to set the values in the `values.yaml`file respective to the variables.

````yaml

      - run: 
          name: migrate database 
          command: |
            kubectl exec deployment/app -n prod -- node_modules/.bin/sequelize db:migrate --env production
  ````

### We seed the backend database using the db:migrate command. 




## Adding a approval stage between environments

Add the approval stage in the workflow, this will pause circleci build and will only proceed further when a approved signal is given.

### In your workflow , add the deploy prod job.

Edit the workflow to include the `deploy-prod ` run in the workflow.


````yaml

workflows:
  version: 2
  build-test-package:
    jobs:
      - build
      - integration:
          requires: 
             - build
      - e2e:
          requires:
            - integration
      - package:
          requires:
            - e2e
      - deploy-test:
          requires:
            - package
      - approval:
          type: approval
          requires: 
            - deploy-test

````
### Here, you can see the job àpproval which requires deploy-test to be approved by you before proceeding to deploy the production environment application

````yaml
      - deploy-prod:
          requires:
            - approval
          filters:
            branches:
              only: master

````
### In your workflow , add the deploy prod job.

Push up to your Git repository:
`````
git add .
git commit -m "deploy test"
git push

`````

### You should see this once it successfully runs:

![Alt text](/screenshots/approval.png?raw=true )


### Here is  how it will look once the run is complete

![Alt text](/screenshots/deployprodjob.png?raw=true )

![Alt text](/screenshots/prodmigration.png?raw=true )


## Integrating logging into AWS CloudWatch