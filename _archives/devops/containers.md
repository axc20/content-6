# Containers, Kubernetes

- docker compose - run it configured locally
- helm - run it configured remotely
- Dockerfile describes blueprint of your image - it is a **custom runtime environment for your app**
- Each image can be built using a base image and further customized (e.g. copy build folder and run at start) which then results in a new image.
- Docker adds a virtualization layer, and images run isolated from host machine. Additonal configuration is needed for all resources that should be available from host system (e.g. folder/file to store logs, network connections, ports mapped to host machine, files mounted to container to be able to persist data, env variables). Containers start with a clean slate every time.
- Useful when scalability and resilience are very important
  - If your web server for frontend is over load threshold, container platform can spin up another container of your image behind a load balancer
  - If your container crashed, container platform can restart it automatically

## Container Orchestration Systems

Container orchestration systems are tools that automate the deployment, scaling, and management of containerized applications. As containers became popular for packaging and deploying applications, the need for managing multiple contianers, often distributed across multiple machines became evident.

### Kubernetes (K8s)

- Open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications
- Features:
  - **Pods**: The smallest deployable units in Kubernetes that can be created and managed. A pod can host multiple containers that form a single unit of deployment.
  - **Services**: An abstract way to expose an application running on a set of pods as a network service.
  - **ReplicaSets and Deployments**: Ensure that a specified number of pod replicas are running at any given time.
  - **Auto-scaling**: Automatically scales the number of pods based on CPU usage or other select metrics.
  - **Volume & Storage**: Allows for persistent storage, supporting multiple storage backends.
  - **Networking**: Provides unique IP addresses for pods and services, with DNS names for these IPs.
  - **Loading balancing**: Distributes network traffic to provide a single IP address to the outside world and send incoming traffic to multiple pod replicas.

### AWS Fargate

- Serverless compute engine for containers that works with Amazon Elastic Container Service (ECS) and Amazon Elastic Kubernetes Service (EKS). With Fargate, you don't need to provision, configure, or scale clusters of virtual machines to run containers.
- AWS Fargate is not an orchestration system by itself. It's a compute engine that can be used with orchestration systems like ECS or EKS. The orchestration part is handled by ECS or EKS, while Fargate takes care of the compute aspect without the need to manage underlying infrastructure.
- Features:
  - **Serverless**: No need to manage the underlying EC2 instances.
  - **Isolation**: Each task or pod runs in its own isolated compute environment.
  - **Scaling**: Automatically scales the compute capacity.
  - **Integrated with AWS**: Easily integrates with other AWS services like VPC, CloudWatch, IAM, etc.

## Links

- https://www.dbooks.org/from-containers-to-kubernetes-with-nodejs-0999773054/
