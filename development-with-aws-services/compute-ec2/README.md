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

### Types

* can generally be categorised based on the number of vCPUs (virtual Central Processing Units) they have, RAM, along with their intended use cases and _cost_. E.g. from t3.micros to m5.8xlarge 1 vCPU vs 32 vCPU and RAM
* We can use same family ( like t, m ) but increase vCPU and RAM

### Bootstrapic Instance t3.micro + linux

When we create a web server for EC2 instance in **Networking** also allow for HTTP internet traffic &#x20;

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

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





