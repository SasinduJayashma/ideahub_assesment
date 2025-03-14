README - Deploying a Health Endpoint

Docker Image is uploade to Docker Hub -> [https://hub.docker.com/r/jummy98/ideahub_webserver/tags]

# ideahub_assesment
This is an assessment provided by Ideahub to owcase the devops skills

## 1.Introduction

This assignment gives a simple Health Check API which runs in a Docker container on an EC2 instance. 


GET /status
Response: {"status": "ok"}


## 2. How to Build and Run the Service

Inside the task folder it has following files,
  * app.py
  * Dockerfile
  * requirements.txt

Background requirments, 

AWS EC2 instance with Docker installed
`sudo yum install -y docker`

Steps to Run

### Clone the repository
`git clone https://github.com/SasinduJayashma/ideahub_assesment.git`
`cd ideahub_assesment`

### Build the Docker image
`docker build -t web-server .`

### Run the container
`docker run -d --name webserver-app -p 8080:80 web-server`

### Verify the endpoint
`curl -s http://localhost:8080/status`


## 3. How to Set Up Jenkins (If Implemented)

Jenkins would automate the build, deployment, and verification of the API. We can write stage to automate all the insfrstuctre and the docker part as well


### Install Jenkins
After intalling we can access the jenkins by public ip with the port we added for the Jenkins (ex: 8081)

### Create Jenkins Pipeline

If Jenkins was used, we have to write a Jenkinsfile with all the stages added, the following Jenkinsfile is an example that would automate building and running the service:

```
pipeline {

    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/SasinduJayashma/ideahub_assesment.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                 'docker build -t web-server .'
            }
        }
        
        stage('Run Docker Container') {
            steps {
                 'docker run -d --name webserver-app -p 8080:80 web-server'
            }
        }

        stage('Verify Service') {
            steps {
                 'curl -s http://localhost:8081/status'
            }
        }
    }
}
```


## 4. How to Use Terraform (If Implemented)
We can use terraform to automate another stage in this application, which is the infrastructure provisioning part and also we could use AWS ECS instead of EC2, so we can utilize a serverless architecute. 


### Install Terraform
We can get the terraform zip file from Hashicorp and install it and finish setuping it!

### Create Terraform Configuration
Following example shows how to setup an AWS ECS on AWS on region us-east-1

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_ecs_cluster" "my_cluster" {
  name = "ideaHub-webserver-cluster"
}

resource "aws_ecs_task_definition" "webserver_task" {
  family                   = "webserver"
  container_definitions    = jsonencode([{
    name      = "webserver-container"
    image     = "jummy98/ideahub_webserver:latest"
    memory    = 512
    cpu       = 256
    essential = true
    portMappings = [{
      containerPort = 8080
      hostPort      = 80
    }]
  }])
}

resource "aws_ecs_service" "webserver_service" {
  name            = "webserver-service"
  cluster         = aws_ecs_cluster.my_cluster.id
  task_definition = aws_ecs_task_definition.webserver_task.arn
  desired_count   = 1
  launch_type     = "FARGATE"
}
```

### Deploy with Terraform

`terraform init`
`terraform apply`


## 5. CloudWatch Logging Setup

We can store all the logs on AWS Cloudwatch, following are the basic steps,

### Install and Configure CloudWatch Agent

    * Install Cloudwatch Agent
    * Configure the agent
    * Start CloudWatch Agent

## 6. How to Access Logs

Container Logs can be accessed by 

`docker logs webserver`

CloudWatch Logs can be accessed by 

`aws logs tail /aws/ec2/webserver-logs --follow`

## 7. Scaling the Service

We can do this with several options, 

### Manual Scaling (Docker)

To scale the service manually we can use following docker commands,

`docker run -d --name webserver-2 -p 8082:81 web-server`

This runs pop up another instance container on port 8082.

### Scaling with ECS (If Terraform was used)

 If we use AWS ECS the follwoing can scales to 3 instances automatically.

`aws ecs update-service --cluster webserver-container --service webserver-service --desired-count 3`

### Scaling with Kubernetes (If Kubernetes was used)

Instead of manually running multiple Docker containers, Kubernetes can be used for orchestration and scaling. 

  * Install Kubernetes - For this implemetation we could use either Minikube for Local or EKS for AWS
  * Create Kubernetes Deployment - For the K8s deployment part we have to create both deployment and service kind, following are basic deployment examples,

  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: ideahub-webserver-app
  spec:
    replicas: 2  # Number of pods to run
    selector:
      matchLabels:
        app: ideahub-webserver
    template:
      metadata:
        labels:
          app: ideahub-webserver
      spec:
        containers:
        - name: webserver-container
          image: jummy98/ideahub_webserver:latest
          ports:
          - containerPort: 80
  ```

  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: webserver-service
  spec:
    selector:
      app: ideahub-webserver
    ports:
    - protocol: TCP
      port: 80
      targetPort: 80
    type: LoadBalancer
  ```

  * Deploy to Kubernetes - `kubectl apply -f deployment.yaml`
  * Scale the Application - `kubectl scale deployment ideahub-webserver-app --replicas=5`