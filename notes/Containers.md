# Docker Containers Management on AWS

* amazon elasitc container service (ECS)
* amazon elasitc kubernetes service (EKS) (open-sourced)
* AWS Fargate
    - serverless contianer platform
    - works with ECS and EKS
* Amazon ECR (elastic container registry)
    - private repository
    - public: amazon ECR Public Gallery https://gallery.ecr.aws

---

Note for cluster
> cluster can have ec2 or fargate launch options

---

ecs - ec2 launch type

* we must provision and maintain the infrastructure (ec2 instances)
* each ec2 instance must run the ecs agent to register in the ecs cluster
* aws can take care for starting/stopping containers

ecs - fargate launch type

* we dont privision anything. its serverless!!!!
* we create task definitions (docker image configurations)
* aws runs ecs tasks (containers) based on cpu/ram we need
* to scale, we just increase number of tasks (containers).

ecs - iam roles for ecs

* ec2 instance profile (for ec2 launch type)
    - used by ecs agent
    - makes api calls to ecs service
    - send container logs to cloudwatch logs
    - pull docker images from ecr

* ecs task(container) role (for both ec2 launch type and fargates)
    - each task(container) can have specific role
    - different roles for different ecs services
    - task role is defined in the task definition!

ecs - load balancer integrations

* application load balancer is supported and work for most use cases for both ec2 and fargate launch types

* network load balancer is recommended only for high thorughput/ high performance use cases

* classic load balancer is supported but NOT recommended. Also its NOT for fargates

ecs - data volums (efs)

* mount efs file systems onto ecs tasks (containers)
* works for both ec2 and fargate launch types
* tasks (containers) running in any AZ will share same data in EFS (because its network volume)
* use cases: persistent multi-AZ shared storage for our containers
* S3 can NOT be mounted as a file system

---

ecs service auto scaling

* automatically increase/decrease the desired number of ecs tasks (containers)

* amazon ecs auto scaling uses aws application auto scaling (not alb)
    - ecs service average cpu utilization, memory utilization, ALB Requests count per target (metric coming from the ALB)

* Target tracking: scale based on cloudwatch metric
* step scaling: scale based on specified cloudwatch alarm
* scheduled scaling: scale based on date/time (predictable changes)

* ecs service auto scaling (task level) is DIFFERENT with EC2 auto scaling (ec2 instance level)

* fargate auto scaling is much easier to setup because its SERVERLESS!!!

--

ec2 launch type -auto scalign ec2 instances

* auto scaling group scaling based on cpu utilization, also it adds ec2 intances over time

* ecs cluster capacity provider
    - used to automatically provision and scale infrastructure for our ecs tasks (containers)
    - capacity provider is paired with an auto scaling group
    - can add ec2 instances when missing capacity (cpu, ram, etc.) 

