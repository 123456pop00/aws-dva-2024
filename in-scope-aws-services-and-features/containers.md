# Containers



* **ECS cluster** with the EC2 launch type is a logical grouping of EC2 instances (1-10 container instances) that ECS can use to run tasks and services.
* **Service** (Service ≠ Container ) :cook: is an **orchestration** mechanism that manages group of identical **tasks**. It ensures that a specific number of  **tasks** (which contain your containers) are always running.&#x20;
  * If you specify 3 tasks in a service, ECS ensures 3 containers are running, it keeps track of task failures, and automatically replaces them as needed to maintain the desired number of running tasks.
  * If one task crashes, ECS will automatically restart it.
* **EC2 IAM Role**: One role for all EC2 instances in your cluster (manages ECS Agent).
* **ECS Task Roles**: Defined <mark style="background-color:red;">per task definition,</mark> provides fine-grained permissions to the containers in that task.
* **Task Definition** blueprint that defines the container :receipt::cook: it describes the task requirements (CPU, Memory, container images, networking type, IAM, etc.)
*   **Task** is the actual unit that is running container :cookie: (or set of containers) based on the task definition. A single task definition can define **up to ten (10) containers**. This allows you to group multiple containers that work together as part of a single application or service





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
  * ASG is visible and configurable in the **EC2 console**, giving full control over the lifecycle and scaling of EC2 instances.
  * You are **responsible** for ensuring the EC2 instances have sufficient capacity (CPU/memory) to run all the ECS tasks.
  * If you scale up tasks but don’t have enough EC2 instances, some tasks will remain in a **PENDING** state until more instances are provisioned.

:cowboy:  **IAM Roles for ECS**

* EC2 instance profile -> used By ECS Agent to make API calls to ECS service, send CloudWatch logs, to pull Docker images, SSM reference

:writing\_hand: **Task definition** is like a **blueprint** for container run app.

* It specifies the **Docker image**, the CPU/memory requirements, environment variables, networking, ports.
* It’s not tied to the cluster directly—it’s just the "instructions" ECS uses when launching a task or service.

:card\_box: **Task placement + Placement strategies**

When task is started ECS must know on which EC2 instance to place it.

1. **Resource Availability**:
   * The instance must have enough **CPU** and **memory** to meet the requirements specified in the task definition.
2. **Task Placement Strategies** (Optional):
   * Control how ECS distributes tasks across instances:
     * **binpack**: Place tasks on the instance <mark style="color:blue;">with the least available CPU/memory to optimize resource usage. Cost saving, we max out instance.</mark>
     * **random**: Randomly select an instance.
     * **spread**: Distribute tasks evenly across instances, AZs, or custom attributes
3. **Task Placement Constrains** (Optional):
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
* We only create ECS Task, based on CPU, memory (RAM) we need, **no EC2 instances will be created in our account**
* We treat each Task <mark style="background-color:red;">in fargate as a</mark> <mark style="background-color:red;"></mark><mark style="background-color:red;">**self-contained unit**</mark> <mark style="background-color:red;"></mark><mark style="background-color:red;">in terms of networking, much like an individual EC2 instance</mark>. As each task gets its own **ENI** with a private IP address (and optionally a public IP if configured) in specified VPC (inside Task definition -> ). We control access to the task using SGs, it is applied at the **ENI level**, which means it controls traffic directly for the task.

```jsonp
"networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": [
        "subnet-12345678",  // VPC subnet ID (e.g., private subnet)
        "subnet-87654321"   // Optional second subnet (if running across multiple subnets)
      ],
       "securityGroups": [
        "sg-12345678"  // Security group ID
      ],
      "assignPublicIp": "ENABLED"  // Optional: Whether to assign a public IP for internet access
    }
  }
}

```



*   You don’t manage EC2 instances or ASGs. AWS fully abstracts and provisions the underlying compute resources required for your ECS tasks.

    * EC2 instances that Fargate uses are not visible in the **EC2 console**.
    * You define **Service Auto Scaling policies to scale tasks directly to ASG for EC2 instances**.&#x20;



:card\_box: **Task placement + placement statgies**

Fargate handles resource allocation and placement based on what we define in task’s CPU and memory requirements i.e. **task definition.** But since we **don’t define EC2 instance-specific parameters** (like instance type) we **can't implement same task placement constrains**.&#x20;

<mark style="color:blue;">**Task placement constraints**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">such as affinity and spreading</mark> <mark style="color:blue;"></mark><mark style="color:blue;">**rely on the concept of placing tasks on specific EC2 instances**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">within an ECS cluster.</mark> Fargate, however, is serverless, so there are **no specific EC2 instances** you are responsible for.
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

### ECS Task: the smallest deployable unit of work for EC2 Launch type & Fargate

ECS task definitions provide a cloud-native alternative to Docker Compose or Dockerfiles.

Task Definitions (JSON) -> metadata / **blueprint** that specifies how Docker containers for ECS should run. Up to 10 containers per task definition.

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
     * **Container Port**:  port your application listens to inside the container (e.g., `80`).
     * **Host Port:** port number on the **EC2 instance** (host) that forwards traffic to the container running on it.
       * To run same  Docker container on the same EC2 container instance set host port = 0 (or empty) to allows multiple containers of the same type to launch -> This avoids conflicts and allows **multiple containers** of the same type to run on the same EC2 instance without fighting over the same port.
         * When we define **only container port and leave&#x20;**<mark style="color:blue;">**host as 0 or empty we get Dynamic Host Port Mapping**</mark>
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
   * **Container-Level Memory (Optional)**:
     * Each container in the task can have:
       * A **hard limit** (`memory`): The maximum amount of memory the container can use.
       * A **soft limit** (`memoryReservation`): The minimum amount of memory reserved for the container.
     * **Task-Level Memory (Required in Fargate)**:
       * For Fargate, the task-level memory must be explicitly set and must align with the CPU-to-memory ratios.
4. **Compatibility**:
   * Specifies whether the task works with **Fargate**, **EC2**, or both.
5. **Versioning**:
   * Each update creates a new **revision** to manage changes easily.

### Sample Task&#x20;

**Network Mode**: `awsvpc` is required for Fargate, allowing each task its own elastic network interface.

#### **How Dynamic Port Mapping Works in ECS with EC2 Launch Type**

1. **Shared ENI for Tasks**:
   * All tasks running on an EC2 instance share the **ENI (Elastic Network Interface)** of that instance.
   * This means all tasks share the same IP address (the EC2 instance’s IP).
2. **Dynamic Port Assignment**:
   * Each container within a task requires a **unique host port** to avoid conflicts.
   * Instead of manually assigning specific ports for each container, ECS dynamically assigns an available port from a predefined range (usually `32768–61000`).
3. **Mapping Between Host and Container Ports**:
   * The container in your task has an **internal container port** (e.g., port `80` for a web app).
   * ECS maps this internal container port to an **available host port** on the EC2 instance (e.g., port `32769`).
   * Th<mark style="background-color:red;">e dynamic mapping ensures multiple tasks can run on the same EC2 instance without port conflicts.</mark>
4. **Routing Traffic**:
   * When a request comes to the EC2 instance's public IP address and a specific host port (e.g., `:32769`), ECS routes it to the corresponding container port (e.g., port `80`) inside the container.

<table><thead><tr><th width="191">Feature</th><th>EC2 Dynamic Port Mapping</th><th>Fargate with ENI</th></tr></thead><tbody><tr><td><strong>IP Address</strong></td><td>Shared (tasks share the EC2's IP)</td><td>Each task gets its own IP (unique)</td></tr><tr><td><strong>Port Usage</strong></td><td>Dynamic host port mapping, Port mapping is used to expose container ports to the host (e.g., <code>8080:80</code> maps container port <code>80</code> to host port <code>8080</code>).</td><td>Each task has unique private IP. Look for ENIs with descriptions like <code>ECS task: &#x3C;task-id></code>. These correspond to your Fargate tasks.</td></tr><tr><td><strong>Security Groups</strong></td><td>Shared at the EC2 level</td><td>Dedicated per task</td></tr><tr><td><strong>Isolation</strong></td><td>Tasks are less isolated</td><td>Full network-level isolation</td></tr><tr><td><strong>Management</strong></td><td>Requires managing EC2 instances</td><td>Fully managed by AWS (serverless)</td></tr></tbody></table>

```jsonp
{
  "family": "example-task-definition", // Name of the task definition family (logical group of versions)
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole", // IAM role used by the task to access AWS resources
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole", // IAM role for pulling container images and logging
  "networkMode": "awsvpc", // Networking mode for the task (required for Fargate)
  "containerDefinitions": [ // List of containers in the task
    {
      "name": "app-container", // Name of the container
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest", // Docker image to use
      "cpu": 256, // CPU units for this container (relative to task CPU)
      "memory": 512, // Hard memory limit in MiB
      "memoryReservation": 256, // Soft memory reservation in MiB
      "essential": true, // Marks this container as essential; the task fails if this container stops
      "portMappings": [ // Port mappings for container-to-host communication
        {
          "containerPort": 80, // Port exposed on the container
          "hostPort": 80, // Port exposed on the host (EC2) or ENI (Fargate)
          "protocol": "tcp" // Protocol used (TCP or UDP)
        }
      ],
      "environment": [ // Environment variables passed to the container
        {
          "name": "ENV_VAR_NAME",
          "value": "value"
        }
      ],
      "secrets": [ // Secrets passed to the container from AWS Secrets Manager or Parameter Store
        {
          "name": "SECRET_NAME",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:my-secret"
        }
      ],
      "logConfiguration": { // Logging configuration for the container
        "logDriver": "awslogs", // Log driver (e.g., awslogs for CloudWatch)
        "options": {
          "awslogs-group": "/ecs/example-task", // CloudWatch log group name
          "awslogs-region": "us-east-1", // AWS region for CloudWatch logs
          "awslogs-stream-prefix": "app" // Prefix for log stream names
        }
      },
      "mountPoints": [ // Data volumes mounted to the container
        {
          "sourceVolume": "my-volume", // Name of the volume
          "containerPath": "/data", // Path inside the container where the volume is mounted
          "readOnly": false // Whether the volume is read-only
        }
      ],
      "volumesFrom": [ // Volumes shared with other containers
        {
          "sourceContainer": "another-container",
          "readOnly": true
        }
      ]
    }
  ],
  "volumes": [ // Volumes available to the task
    {
      "name": "my-volume", // Logical name of the volume
      "host": { // Host volume (for EC2 instances only)
        "sourcePath": "/path/on/host"
      }
    },
    {
      "name": "efs-volume", // EFS volume (shared storage across tasks)
      "efsVolumeConfiguration": {
        "fileSystemId": "fs-12345678", // EFS filesystem ID
        "rootDirectory": "/", // Root directory within the filesystem
        "transitEncryption": "ENABLED" // Whether to use transit encryption
      }
    }
  ],
  "placementConstraints": [ // Constraints for task placement
    {
      "type": "memberOf",
      "expression": "attribute:ecs.instance-type =~ t2.*" // Example: Restrict tasks to specific instance types
    }
  ],
  "requiresCompatibilities": [ // Launch types supported by this task
    "EC2",
    "FARGATE"
  ],
  "cpu": "512", // Task-level CPU allocation (required for Fargate)
  "memory": "1024", // Task-level memory allocation (required for Fargate)
  "runtimePlatform": { // Specify OS and architecture (useful for multi-architecture tasks)
    "operatingSystemFamily": "LINUX", // LINUX or WINDOWS
    "cpuArchitecture": "X86_64" // X86_64 or ARM64
  },
  "tags": [ // Tags for categorizing and managing tasks
    {
      "key": "Environment",
      "value": "Production"
    }
  ],
  "proxyConfiguration": { // (Optional) Proxy configuration for the task
    "type": "APPMESH",
    "containerName": "envoy", // Name of the proxy container
    "properties": [
      {
        "name": "ProxyEgressPort",
        "value": "15001"
      },
      {
        "name": "AppPorts",
        "value": "8080"
      }
    ]
  },
  "ephemeralStorage": { // Additional ephemeral storage (Fargate only)
    "sizeInGiB": 21 // Size in GiB (default is 20 GiB, max is 200 GiB)
  }
}

```





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

* Target tracking - specific cloudwatch&#x20;
* Step Scaling
* Scheduled Scaling ( pattern)

#### How to scale with ECS ?

* Add EC2 instances inside ASG based on **CPUUtilization**&#x20;
* **ECS Cluster Capacity Provider -** automatically scales the infrastructure for ECS tasks \*\*\* better for ECS



<details>

<summary>ECR</summary>

To pull a Docker image from a **private Amazon ECR repository**

1. Login / Authenticate Docker to ECR `aws ecr get-login-password`` `<mark style="color:red;">`--region $aws_region`</mark>` ``| docker login --username AWS --password-stdin`` `<mark style="color:red;">`$ecr_url`</mark>
2. Pull the Docker Image `docker pull`` `<mark style="color:red;">`$ecr_docker_iamge_url`</mark>

:exclamation: **Authentication and pulling must happen in the same region**. You can't auth in one region and pull image from another.

* Authenticate Docker to your ECR:
*   **Authentication**: The `aws ecr get-login-password` command gets the token for **one region**, and the token is specific to that region.

    ```bash
    aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.eu-north-1.amazonaws.com
    ```
*   Pull the image:

    ```bash
    docker pull 123456789012.dkr.ecr.eu-north-1.amazonaws.com/my-app-repo:latest
    ```

</details>



#### Useful Links:

* [https://gallery.ecr.aws/](https://gallery.ecr.aws/)
* [https://ecsworkshop.com/microservices/crystal/](https://ecsworkshop.com/microservices/crystal/)
* [https://explore.skillbuilder.aws/learn/course/external/view/elearning/14608/amazon-eks-for-developers-online-course-supplement](https://explore.skillbuilder.aws/learn/course/external/view/elearning/14608/amazon-eks-for-developers-online-course-supplement)
* [https://catalog.workshops.aws/ecs-immersion-day/en-US/10-about-ecs](https://catalog.workshops.aws/ecs-immersion-day/en-US/10-about-ecs)
