# Containers



{% hint style="info" %}
Dockerfile - single service

:warning: Always check we create an Dockerfile inside **dir** of our application + .dockerignore (for \*.md node\_modules etc)

:construction\_site: Docker build for single service&#x20;

Simple node server&#x20;

```dockerfile
FROM node:23-alpine
RUN npm install -g nodemon
WORKDIR /app
COPY package.json .
RUN npm install 

COPY . .

EXPOSE 5000

CMD [ "npm", "run", "dev" ]

```

**Build ->** docker build -t jtorp/express-dummy-api .&#x20;

_if pushing to Hub username+image name tag will be :latest if not specified_

**Run ->** `docker run -d --name c1 -p 5000:5000 --rm imagename:tag`

**Stop** -> `docker ps then docker stop c1 or`` `_`id`_

**Remove after stop ->** `docker start --name c1  -p 5000:5000`` `**`--rm`**` ``imagename:tag`&#x20;

**Remove volumes after stop->** `docker run -d --name c1 -p 5000:5000 --rm -v`&#x20;



Map project folder on local to the container (Volumes) to track dev changes and not restarting the server, changes are picked up

`/Users/me/Documents/2024/Docker/myapp:/app imagename:tag`



Docker Compose - for multiple containers / services&#x20;



```yaml

services:
  api:
    build: ./api
    image: myapp:latest
    container_name: api-c
    ports:
      - 5000:5000
    restart: always
    volumes:
      - ./api:/app                # Maps the ./api directory on the host to /app in the container
      - /app/node_modules         # Prevents overwriting the container's node_modules
  redis:
    image: redis
    
```

`docker compose up`

`docker compose down`&#x20;
{% endhint %}

{% tabs %}
{% tab title="ECS Launch Type" %}
* Docker management platform&#x20;
* EC2 Launch Type: provision and maintain infrastructure
* Each EC2 instance must run **ECS Agent** that will register in the ECS Cluster
* Each Docker container will be placed accordingly onto the EC2 instance within a **Cluster**
* You define and manage an **Auto Scaling Group (ASG)** for EC2 instances that make up your ECS cluster.
  * ASG is visible and configurable in the **EC2 console**, giving you full control over the lifecycle and scaling of these EC2 instances.
  * You are **responsible** for ensuring the EC2 instances have sufficient capacity (CPU/memory) to run all the ECS tasks.
  * If you scale up tasks but don’t have enough EC2 instances, some tasks will remain in a **PENDING** state until more instances are provisioned.

:cowboy:  **IAM Roles for ECS**

* EC2 instance profile -> used By ECS Agent to make API calls to ECS service, send CloudWatch logs, to pull Docker images, SSM reference

:card\_box: **Task placement + placement strategies**

When task is started ECS must know where ( on which EC2 instance ) to place it.

1. **Resource Availability**:
   * The instance must have enough **CPU** and **memory** to meet the requirements specified in the task definition.
2. **Task Placement Strategies** (Optional):
   * Control how ECS distributes tasks across instances:
     * **binpack**: Place tasks on the instance <mark style="color:blue;">with the least available CPU/memory to optimize resource usage. Cost saving, we max out instance.</mark>
     * **random**: Randomly select an instance.
     * **spread**: Distribute tasks evenly across instances, AZs, or custom attributes
3. **Task Placement Constrains (**Optional):
   * You can define constraints in the task definition or ECS service to restrict where tasks can run, such as:
     * **Instance Type**: e.g., tasks can only run on `t3.large` instances.
     * **Availability Zone**: Place tasks in specific AZs
     * Custom Attributes: for example, `memeberOf`to only run on EC2 instances that match specific criteria.
       * To isolate workloads by environment (e.g., dev, staging, production) when you have a single ECS cluster managing multiple environments. (e.g., `environment=prod` or `type=compute-optimized`), you can use `memberOf` to ensure tasks only run on instances matching those attributes.
       *   Place tasks on specific EC2 instances based on hardware attributes (e.g., instance types, GPU availability, storage types).&#x20;

           ```
             "expression": "attribute:ecs.instance-type == t3.micro || attribute:ecs.instance-type == t3.small || attribute:ecs.instance-type == t3.medium"
           ```
       * To divide workloads by application types or roles (e.g., front-end, back-end, batch jobs) within the same cluster, e.g. `attribute:role=frontend`
     * **`distinctInstance`** enforces placement logic directly, to achieve fault tolerance and prevent multiple tasks from being co-located on the same EC2 instance.
{% endtab %}

{% tab title="Fargate Launch Type - Serverless" %}
* Fargate + EFS = ultimate serveless combination
* **No EC2 instances to manage,** Fargate will scale according to your ASG policies **but will not upgrade instance type -** it will scale **horizontally**&#x20;
* We only create ECS Task, based on CPU, RAM we need, **no EC2 instances will be created in our account**
*   You don’t manage EC2 instances or ASGs. AWS fully abstracts and provisions the underlying compute resources required for your ECS tasks.

    * EC2 instances that Fargate uses are not visible in the **EC2 console**.
    * You define **Service Auto Scaling policies to scale tasks directly to ASG for EC2 instances**.&#x20;



:card\_box:**Task placement + placement statgies**

EC2 instances are managed by you, so you don’t worry about placement
{% endtab %}

{% tab title="ECS Task Role" %}
* For both ECS Type & Fargate -> <mark style="color:blue;">define in the task definition</mark>
* Allow to create a **specific role per task** ( Task A => Task A Role to run API calls to S3, Task B Role to make calls to RDS etc...)


{% endtab %}
{% endtabs %}

### ALB integration

We can interface ECS Cluster with ALB ( which will have inbound on a port 80/443 Security Group )

#### ECS Launch Type

* Must allow on ECS instance SG **any port** from the ALB SG -> because of <mark style="color:blue;">Dynamic Host Port Mapping</mark>
* In ECS, the security group for the ECS instances should allow inbound traffic **only from the ALB security group** (referencing the ALB SG). This ensures that the instances are not directly accessible from the internet.
*   An ECS cluster using the EC2 launch type typically relies on an **Auto Scaling Group (ASG)** of EC2 instances. These instances are visible in the **EC2 console**, and you have full control over their lifecycle.


* When defining the Cluster and chosoing to create ASG -> this will be created by cluster

<figure><img src="../.gitbook/assets/Screenshot 2024-11-16 at 13.31.47.png" alt=""><figcaption></figcaption></figure>

#### Fargate Launch Type

* Each task is assigned its own **Elastic Network Interface (ENI)**. so we only define in ( Task definition) a container port.&#x20;
  * Fargate eliminates the need for host port configuration. You don’t specify a **host port** because there’s no shared underlying host for the containers; AWS handles this infrastructure behind the scenes.
  * This ENI has a unique private IP address, and the task is treated as an independent networking entity in the VPC.
* ECS ENI Securoty Group must allow port 80 from ALB
* With Fargate, you don’t define ASG rules like you do with EC2 instances. Instead, you define **ECS Service Auto Scaling policies for your tasks** (not instances), such as specifying a minimum and maximum number of tasks to run.&#x20;

NLB is recommended for high throughput applications or to pair with Private Link

### ECS Task: the smallest deployable unit of work in ECS.

ECS task definitions provide a cloud-native alternative to Docker Compose or Dockerfiles.

Task Definitions (JSON) -> metadata / blueprint that specifies how Docker containers for ECS should run. Up to 10 containers per task definition.

* **What to run**: Docker images, ports, and commands.
* **How to run it**: Resources, roles, and network settings.
* **Where to store data**: Volumes and persistent storage.

#### **When ECS Task = Docker Compose**

When your ECS **task definition** includes **multiple containers** running as a unit (e.g., web app + logging agent), it’s directly analogous to a **Docker Compose service definition** where multiple containers are configured to run together.

#### **When ECS Task = Dockerfile**

If your ECS task runs a **single container**, then the **Dockerfile** is equivalent to the container-specific configuration (image + build context), and the ECS task definition adds extra runtime configuration (e.g., CPU, memory, networking).

#### **Core Components of a Task Definition** :white\_check\_mark:

1. **Container Definitions**:
   * Specifies the Docker images (e.g., `nginx:latest` or a custom ECR image).
   * Resource requirements (CPU, memory) for container
   * Port mappings `"containerPort": 80, "hostPort": 80`
   * Environment variables passed into the container ( any sensitive variables typically in .env) -> use SSM Parameter Store or Secrets Manager to fetch them at run time within task
     * Bulk evn variables loading - fetching from S3
   * Logging, and health checks for each container in the task
2. **Task-Level Settings**:
   * **Task Networking Mode**: Controls how containers interact with the network (`awsvpc` recommended for Fargate, `bridge` or `host` for EC2).
   *   **Volumes**: Defines shared or persistent storage

       such as:

       * **EFS Volumes**: Persistent, shared storage.
       * **Ephemeral Storage**: Temporary, task-scoped storage. Fargate tasks 20 - 200GiB
       * **Bind Mounts**: Directories on the host EC2 instance. Application container write to shared bind mount storage and metrics container can read from it. _Sidecar container pattern_
   * **IAM Roles**:
     * ECS Task role per Task definition - service will assume the role
     * Task Role: Grants the task permissions to AWS services (e.g., S3, DynamoDB).
     * **Execution Role**: Lets ECS pull images and push logs to CloudWatch.
3. **Resource Requirements**:
   * **CPU and Memory**:
     * For **Fargate**, you define task-level CPU and memory requirements.
     * For **EC2**, containers can share resources from the host.
4. **Compatibility**:
   * Specifies whether the task works with **Fargate**, **EC2**, or both.
5. **Versioning**:
   * Each update creates a new **revision** to manage changes easily.



### Data Volumes&#x20;

**EFS**&#x20;

* Compatible with ECS + Fargate Launch Types
* EFS is a popular choice for  Multi-AZ persistent shared storage for containers&#x20;

**Ephemeral Storage (Task Storage)**:

* Every Fargate task gets ephemeral storage (up to 200 GiB as of 2023), is **lost when the task stops**.
* For EC2 instance ephemeral storage is tied to the  instance itself.

**EBS (Elastic Block Store)**:

* Supported indirectly with ECS launch type because EBS volumes are attached to the EC2 instance running the ECS task and can be mounted to containers.
* Not compatible with Fargate

**S3**

* S3 **cannot** be mounted as a file system - programmatic access only (via SDKs or CLI)



### ECS Auto Scaling

Automatically increase :arrow\_double\_up: / decrease :arrow\_double\_down: ECS tasks

1. CPU Utilisation
2. Memory - RAM
3. ALB count per target

* Target tracking - specific cludwatch&#x20;
* Step Scaling
* Scheduled Scaling ( pattern)

#### How to scale with ECS ?

* Add EC2 instances inside ASG based on **CPUUtilization**&#x20;
* **ECS Cluster Capacity Provider -** automatically scales the infrastructure for ECS tasks \*\*\* better for ECS





#### Useful Links:

[https://gallery.ecr.aws/](https://gallery.ecr.aws/)

[https://explore.skillbuilder.aws/learn/course/external/view/elearning/14608/amazon-eks-for-developers-online-course-supplement](https://explore.skillbuilder.aws/learn/course/external/view/elearning/14608/amazon-eks-for-developers-online-course-supplement)

