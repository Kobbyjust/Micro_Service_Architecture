# ABOUT THIS ARCHITECTURE

This project is a micro service architecture which consists of
1. Amazon ECS - Running on EC2
2. Amazon ECR - 
3. Load Balancer -  To distribute Traffic to the various Micro service component based on Rules

![For a little overview, kindly check the architecture diagram](./Micro_Service_Architecture.jpeg)

# STEP TO CREATE THE PROJECT
We will be using only one component of the Micro services architect as we go through
the various stages of this text instructions BUT  kindly repeat same for the various components

In the step below I will be usings the "posts" services

1. Provision Amazon ECR Repositories https://console.aws.amazon.com/ecs/home?#/repositories to create
the various repository for the various component (posts, users, threads)
-- You can use the  "View push commands" button of the "posts" repository page to access all the necessary commands

2. Build and Push Images For each Services
In the project folder microservices/services you will have folders with files for each service.
You will need access to Docker to build and push the images for each service

 ```aws ecr get-login-password --region [your-region] | docker login --username AWS --password-stdin [aws_accountID].dkr.ecr.[your-region].amazonaws.com```

Build and Tag Each Image
<br>

```docker build -t posts ./posts```
<br>

```docker tag posts:latest [aws_accountID].dkr.ecr.[your-region].amazonaws.com/posts:latest```
<br>

```docker push [aws_accountID].dkr.ecr.[your-region].amazonaws.com/posts:latest```

3. Launch and Deploy Amazon CloudFormation console
This creates and Amazon ECS cluster deployed behind an Application Load Balancer

Amazon Elastic Container Service (Amazon ECS) is a highly scalable, high performance container management service that supports Docker containers and allows you to easily run applications on a managed cluster of Amazon EC2 instances

  - Navigate to the Amazon CloudFormation console https://console.aws.amazon.com/cloudformation/home
  - Create a new stack and upload the template "ecs.yaml" located in microservices_container/infrastructure
  Fill the information below and check the capabilities statement checkbox

4. Writ task definitions for your services
  - Navigate to the Amazon ECS console and selete Task definitions
  - Create a new Task Definition
  - In the Select launch type compatibility page, select the EC2 option and then select Next step
  - In the Configure task and container definitions page, scroll to the Volumes section and select the     Configure via JSON button.
  Copy and paste the following code snippet into the JSON field, replacing the existing code.
  Remember to replace the [service-name], [account-ID], [region], and [tag] placeholders.
  Note: The following parameters are used for the task definition:

    Name = [service-name: posts, threads, and users] 
    Image = [Amazon ECR repository image URL]:latest 
    cpu = 256 
    memory = 256 
    Container Port = 3000 
    Host Post = 0

    ```
    {
    "containerDefinitions": [
        {
            "name": "[service-name]",
            "image": "[account-id].dkr.ecr.[region].amazonaws.com/[service-name]:[tag]",
            "memoryReservation": "256",
            "cpu": "256",
            "essential": true,
            "portMappings": [
                {
                    "hostPort": "0",
                    "containerPort": "3000",
                    "protocol": "tcp"
                }
            ]
        }
    ],
    "volumes": [],
    "networkMode": "bridge",
    "placementConstraints": [],
    "family": "[service-name]"
    } ```
    
    --- Repeat the steps to create a task definition for each service

5. Configure The Application Load Balancer: Target Groups (TG)
  - Navigate to the load balancer at the EC2 console of the https://console.aws.amazon.com/ec2/v2/home?#LoadBalancers:
  - Locate the Load Balancer created by the CloudFormation template
  - In the Description tab, locate the VPC attribute (in this format: vpc-xxxxxxxxxxxxxxxxx)
  
    # Configure the ALB Target Group
    Navigate to the Target Group section of the EC2 Console.
  - Select Create target group.
    Configure the following Target Group parameters (for the parameters not listed below, keep the default values):
    For the Target group name, enter api.
    For the Protocol, select HTTP.
    For the Port, enter 80.
    For the VPC, select the value that matches the one from the Load Balancer description. This is most likely NOT your default VPC.
    Access the Advanced health check settings and edit the following parameters as needed: 
    For Healthy threshold, enter 2.
    For Unhealthy threshold, enter 2.
    For Timeout, enter 5.
    For Interval, enter 6.
    Select Create.




