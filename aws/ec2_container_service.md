# [EC2 Container Service](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)

Official Developer Guide.

## Components of ECS

Cluster
* A logical grouping of container instances that you can place tasks on. For more information, see Amazon ECS Clusters.
* in cluster config, you specify the container instances that your tasks can be placed on, the address range that you can reach your instances and load balancer from, and the IAM roles to use with your container instances that let Amazon ECS take care of this configuration for you.

Container instance
* An Amazon EC2 instance that is running the Amazon ECS agent and has been registered into a cluster. For more information, see Amazon ECS Container Instances.
* When you run tasks with ECS, your tasks are placed on your active container instances
* ECS container agent makes calles to ECS on your behalf, so you need to launch container instances with an IAM role that authenticates to your account and proivded the required resource permission.

Task definition
* A description of an application that contains one or more container definitions. For more information, see Amazon ECS Task Definitions.
* A task definition is like a blue print for your application. Every time you launch a task in Amazon ECS, you specify a task definition so the service knows which Docker image to use for containers, how many containers to use in the task, and the resource allocation for each container.
* A task definition creates a **Service**

Scheduler
* The method used for placing tasks on container instances. For more information about the different scheduling options available in Amazon ECS, see Scheduling Amazon ECS Tasks.

Service
* An Amazon ECS service allows you to run and maintain a specified number of instances of a task definition simultaneously. For more information, see Services.
* a Service is created from a **Task Definition**
* a service launches and maintains a specified number of copies of the task definition in your cluster.
* a task will be restarted if it becomes unhealthy or stops, in order to comply with the service definition

Task
* An instantiation of a task definition that is running on a container instance.

Container
* A Linux container that was created as part of a task.


## [Cleaning up your Amazon ECS Resources](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CleaningUp.html)

Scale down services:
```
$ aws --region us-west-2 ecs update-service --cluster default --service service_name --desired-count 0
```

Delete services:
```
$ aws  --region us-west-2 ecs delete-service --cluster default --service service_name
```

Deregister container instances:
```
$ aws --region us-west-2 ecs deregister-container-instance --cluster default --container-instance container_instance_id --force
```

Delete a cluster: AWS Console

Delete the AWS CloudFormation Stack: AWS Console
* Deleting the CloudFormation stack terminates the EC2 instances, removes the Auto Scaling group, deletes any Elastic Load Balancing load balancers, and removes the Amazon VPC subnets and Internet gateway associated with the cluster.

## [Connecting to container instance](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance-connect.html)

The default user for an ECS-optimized AMI (Amazon Machine Image) is `ec2-user`. Connect using this command:
```
$ ssh -i /path/to/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```

## [CloudWatch Logs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_cloudwatch_logs.html)

You can configure container instances to send log information to CloudWatch logs. See this page for more info.

## [Starting a task at container instance launch time](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/start_task_at_launch.html)

Depending on your application architecture design, you may need to run a specific container on every container instance to deal with operations or security concerns such as monitoring, security, metrics, service discovery, or logging.

See this page for more info.

## [Deregister container instance](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/deregister_container_instance.html)

You can deregister container instances from the cluster. This stops the instance from being able to accept new tasks and orphans task running on the instance. Note that deregistering the instance does NOT terminate it.

Note:
> If your instance is maintained by an Auto Scaling group or AWS CloudFormation stack, terminate the instance by updating the Auto Scaling group or AWS CloudFormation stack; otherwise, the Auto Scaling group will recreate the instance after you terminate it.

## [Container Agent](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_agent.html)

The ECS container agent comes installed on all ECS-optimized AMIs, but you can install it manually or configure it, if you'd like. See this topic for more info.

## [Task Definitions](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_defintions.html)

A task definition is required to run Docker containers in Amazon ECS. Some of the parameters you can specify in a task definition include:

* Which Docker images to use with the containers in your task
* How much CPU and memory to use with each container
* Whether containers are linked together in a task
* What (if any) ports from the container are mapped to the host container instance
* Whether the task should continue to run if the container finishes or fails
* The command the container should run when it is started
* What (if any) environment variables should be passed to the container when it starts
* Any data volumes that should be used with the containers in the task
* What (if any) IAM role your tasks should use for permissions

Your entire application stack does not need to exist on a single task definition, and in most cases it should not. Your application can span multiple task definitions by combining related containers into their own task definitions, each representing a single component. For more information, see Application Architecture.

## [Application Architecture](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/application_architecture.html)

* Every container in a task definition must be on the same container instance, so overloading a single task definition to include many different containers limits flexibility in scaling and in resources
* Create task definitions that group the container used for a common purpose, such as linked containers that must be run together
  * ie. add a log streaming container to your front-end service and include that in the same task definition

## [Task Definition Parameters](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html)

Task definitions are split into four basic parts: the task family, the IAM task role, container definitions, and volumes. The family is the name of the task, and each family can have multiple revisions. The IAM task role specifies the permissions that containers in the task should have. Container definitions specify which image to use, how much CPU and memory the container are allocated, and many more options. Volumes allow you to share data between containers and even persist the data on the container instance when the containers are no longer running. The family and container definitions are required in a task definition, while volumes are optional.

This page documents the task definition parameters in detail.

## [Scheduling ECS Tasks](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/scheduling_tasks.html)

ECS provides schedulers for running tasks and containers:
* service scheduler: long-running tasks and applications
* `RunTask` action: for batch jobs or single run tasks that are placed onto the cluster for you
* `StartTask` action: allows you to specify a container instance for the task, to place task manually on a specific container instance, or so you can integration with custom, third-party schedulers

[This article](https://aws.amazon.com/blogs/compute/better-together-amazon-ecs-and-aws-lambda/) describes using ECS `RunTask` in conjunction with SQS and Lambda.

## [Services](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html)

Amazon ECS allows you to run and maintain a specified number (the "desired count") of instances of a task definition simultaneously in an ECS cluster. This is called a service. If any of your tasks should fail or stop for any reason, the Amazon ECS service scheduler launches another instance of your task definition to replace it and maintain the desired count of tasks in the service.

In addition to maintaining the desired count of tasks in your service, you can optionally run your service behind a load balancer. The load balancer distributes traffic across the tasks that are associated with the service.
