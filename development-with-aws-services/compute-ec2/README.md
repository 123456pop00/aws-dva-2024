# Compute: EC2

* **IaaS** on AWS with EC2&#x20;
* VMs rentals - EC2
  * OS: linux, mac, windows
  * CPU cores
  * RAM
* Store data&#x20;
  * **network** attached - **NAS** - storage space connected to your network, where multiple instances can access - EBS(does **not auto-scale**) & EFS drives
  * **hardware** attached EC2 instance store (_this volume  is attached to the underlying physical host, if you stop or terminate your EC2 instance, all data written to the instance store volume will be deleted.)_
* ELB & ASG for HA and Scalability
* Security: Firewall with **Security Group**
* **Bootstrap Script** with User Data, script only runs once at the instance first start
  * to install updates
  * install software
  * upload common files from www
* Each instance **type** offers a different balance of compute, memory, network, and storage resources.
* According Responsibility **custom is responsible** for operating-system patches and updates on EC2 Instances.

## EC2

<details>

<summary>A<strong>WS Nitro System</strong> </summary>

AWS Nitro System is the underlying platform of the next generation of EC2 instances. Nitro **Hypervisor** delivers performance that is _indistinguishable from bare metal_

* Renting VM
  * up to 400 Gbps Ethernet networking.
* Storing on virtual drives EBS
  * local EBS, instance, EFS
* Distributing loads - ELB
  * regions with multiple AZ
  * multiple AZs
* Auto scaling - ASG
  * Dynamic policies
* AWS regularly performs routine hardware, software, and network maintenance with minimal disruption

</details>

### Instance Types

* can generally be categorised based on the number of vCPUs (virtual Central Processing Units) they have, RAM, along with their intended use cases and _cost_. E.g. from t3.micros to m5.8xlarge 1 vCPU vs 32 vCPU and RAM
* We can use same family ( like t, m ) but increase vCPU and RAM
* List of all types[https://instances.vantage.sh/?region=eu-north-1](https://instances.vantage.sh/?region=eu-north-1)

```
m5.2xlarge
| |  |
| |  └─ Size
| └──── Generation (fifth-generation general-purpose instance)
└────── Family 

```

#### **Family (Class)**&#x20;

The **family** (or class) indicates the primary **use case** or **category** of the instance. Each family is optimised for different **workloads** based on factors like CPU, memory, storage, or network performance.

* **Examples of Families**:
  * **General Purpose**: <mark style="color:red;">`m`</mark> (e.g., `m5`), <mark style="color:red;">`t`</mark> (e.g., `t3`) - good balance of performance and efficiency (ie compute, memory & networking).
    * basic tasks like running a small website or development/testing environments/code repositories
    * running small to medium databases, backend servers
  * **Compute Optimized**: <mark style="color:purple;">`c`</mark> (e.g., `c6g`, `c5`) - great for compute-intensive tasks that require high performance processors, ideal for _compute bound applications_ that benefit from high performance processors.
    * Gaming servers
    * media transcoding
    * HPC&#x20;
    * batch workloads
    * LLM, ML
  * **Memory Optimized**: <mark style="color:blue;">`r`</mark> (e.g., `r5`, `r6g, r7a`), <mark style="color:blue;">`x`</mark> (e.g., `x1e`) - great to deliver high performance when large amounts of RAM required, for carrying heavy loads (perfect for memory-intensive applications like databases).
    * Best for **high in-memory workloads** where large memory capacity is essential (e.g., in-memory databases and data analytics)
    * Relational / non-relational DBs
    * Distributed web scale stores (elastic cache)
    * Applications performing real-time processing of unstructured data
    * BI tools with in-memory DBs&#x20;
    * Financial modeling, fraud detection
    * For example  r7a.48xlarge instance is in the memory optimized family with 192 vCPUs, 1536.0 GiB of memory and 50 Gibps of bandwidth is the latest  generation, based on high-performance AMD processors perfect for **memory-intensive** and **highly parallel tasks:** like Redis, Memcached, SAP HANA, ERP systems (oracle), Cache Layers for High-Traffic Applications like real-time bidding, LLM pre-trained models
  * **Storage Optimized**: <mark style="color:purple;">`i`</mark> (e.g., `i3`), <mark style="color:purple;">`d`</mark> (e.g., `d2`) - perfect for heavy lifting for storage-intensive tasks ,that require high, sequential R-W access on local storage
    * Best for **high I/O workloads** that need **low-latency storage**
    * Data warehousing and OLTP (Online Transaction Processing) systems
    * High-performance databases requiring fast disk reads/writes (e.g., Cassandra, MongoDB)
    * Applications needing consistent, high IOPS like financial transactions or real-time bidding platforms
    * I4i instances are designed for high IOPS and low-latency storage, they may be more cost-effective for **I/O-bound applications** where **fast data access is critical**, rather than memory capacity.
  * **Accelerated Computing**: `p` (for GPUs, e.g., `p4`), `g` (for GPUs, e.g., `g4`), `f` (for FPGAs, e.g., `f1`)
    * Best for processing complex algorithms for tasks such as **machine learning training**, **deep learning**, and **neural network modeling** - p series
    * Best for high quality / detail animations or rendering films, needign **powerful graphics workstations** (GPUs) - g series
    * Real-time rendering for video games, movies, or architectural visualizations - g series

**Generation**

The **generation** specifies the **version** or iteration of the instance family. Higher generation numbers generally indicate newer hardware with better performance, improved features, and often more efficient power consumption.

* `m5` is a fifth-generation general-purpose instance, while `m4` is its fourth-generation predecessor.

**Size**

The **size** defines the **compute, memory, and networking capacity** within the family and generation. The size generally scales linearly (e.g., `xlarge`, `2xlarge`, `4xlarge`, etc.), _where each increase in size often doubles the CPU and memory compared to the previous size._

* **Smallest Sizes**: `.micro`, `.small`, `.medium`
* **Standard Sizes**: `.large`, `.xlarge`
* **Larger Sizes**: `2xlarge`, `4xlarge`, `8xlarge`, etc.
* **Very Large Sizes**: Some instance families offer even larger sizes, like `12xlarge`, `16xlarge`, and `32xlarge`.

#### Networking & Security Groups

When we create a web server for EC2 instance in **Networking** also allow for HTTP internet traffic &#x20;

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### Security Groups&#x20;

SG are fundamental of network security in AWS. They are virtual firewalls that controls inbound and outbound traffic for an Amazon EC2 instance.

* &#x20;Only have <mark style="color:green;">**ALLOW**</mark> rule, if it is not explicitly allowed it will be blocked (ie  Deny all inbound traffic by default)
* **Region-specific,** SG for **us-east-1** region, will not be available for instances in the **us-west-2** region
* **VPC-specific,** can only apply a security group to instances that are in the same VPC.
* Rules include IP addresses or reference other security groups IDs
* Rules are are aggregated and evaluated together, **up to 5 SG per instance,** for example have a separate SSH-Access SG
* If one security group allows traffic (e.g., SSH) and another denies it, the **allow rule takes** **precedence**, and the instance will still allow the traffic.
* **Stateful** packet filtering, i.e. <mark style="color:red;">return traffic is automatically allowed</mark>
* SG will check a **list of what is allowed** into the instance, but won’t bother checking the list on the way out, **so all traffic is allowed out**
* We modify  SG to accept a specific traffic, e.g. in case of website we want HTTPS but not other like OS or admin requests, inbound rule to allow anyone when accessing port 443![](<../../.gitbook/assets/Screenshot 2024-11-01 at 16.12.44.png>)
* If there is more than one rule for a specific port, Amazon EC2 applies the most **permissive rule**

<details>

<summary>Ports</summary>

22 = SSH ( Secure shell) log into a Linux instance

3389 = RDP ( Remote Desktop Protocol) - log into Windows instance

21 = FTP ( File transfer protocol) - uploads files into a file share

22 = SFTP ( Secure File Transfer Protocol) - upload files using SSH

</details>



#### Advanced Section

:exclamation: **(Termination protection) -** prevents an EC2 instance from being accidentally terminated. When this feature is enabled, you cannot terminate the instance through the AWS Management Console, CLI, or API until you disable termination protection.

:exclamation:If you stop and start instance **public** IPv4 **can be changed, private stays the same**

* **Be able to delete (terminate) the instance easily**: You should **disable termination protection** when setting up the instance.&#x20;
* **To safeguard against accidental termination**: You should **enable termination protection**.

#### Security Group

* Allow all traffic for outbount 0.0.0.0/0 source
* Allow  Ports 22 and 80 for inbound for TCP from 0.0.0.0/0 source
* when using Public IPv4 address - check it's using **http** or it will not load &#x20;



## SSH into instance

```bash
ssh -i /path/to/your-key.pem ec2-user@your-instance-public-ip
```

Permissions 0644 too open error&#x20;

```bash
#set pem file 
# **400 (`r--------`)**: Owner has read-only permissions, and neither group nor others have any permissions.
# **0644 (`rw-r--r--`)**: Owner has read and write permissions, group and others have read-only permissions.

chmod 0400 name-of-pem-file.pem

```

#### Default Usernames by OS

Here are the common default usernames for various Linux distributions on EC2:

```bash
whoami 
```

* **Amazon Linux**: `ec2-user`
* **RHEL**: `ec2-user` or `root`
* **Ubuntu**: `ubuntu`
* **Debian**: `admin` or `root`
* **CentOS**: `centos`
* **SUSE**: `ec2-user` or `root`

#### Standard Content of home/ec2-user

```
├── home            # Home directories for users
│   └── ec2-user    # Home directory for the 'ec2-user'
│       ├── .bash_history      # History of shell commands
│       ├── .bash_logout       # Commands to run on logout
│       ├── .bash_profile      # User profile configuration
│       ├── .bashrc            # Shell configuration
│       ├── .config            # Application-specific configuration directory
│       │   └── ...            # Application-specific config files
│       ├── .lesshst           # History file for 'less' command
│       ├── .ssh               # SSH-related files
│       │   ├── authorized_keys # Public keys for SSH access
│       │   └── known_hosts     # Known SSH hosts
│       └── ...
```











