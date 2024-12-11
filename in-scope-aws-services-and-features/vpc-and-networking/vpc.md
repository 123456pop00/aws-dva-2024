---
icon: lock
---

# VPC

**VPC** ⇒ virtual private cloud, a private network to deploy resources

* Linked to a specific region
  * Can go across 2 -3 AZs
  * **Subnets inside \~ 3 subnets**
  * Can have **1 IGW**

**Subnet** ⇒ partition of your network, ranges of IP addresses in your VPC to group resources together, defined at AZ level

**CIDR Range** ⇒ range of range of IP addresses that are allowed within your VPC

**Elastic IP** ⇒  fixed public IPv4, ongoing cost if not in-use

**Virtual Private Gateway** ⇒ for private subnets

**NAT Gateway (fully managed)** ⇒ network address translator, translates private IP addresses to a public IP address, ie allow private subnets to access Internet. IPv6 often avoids this by providing a unique address for each device.

**NAT Instance** and a **NAT Gateway** serve the **same primary purpose**: enabling resources in a **private subnet** to access the internet **outbound only** (e.g., for updates, API calls, or downloading files) while keeping those resources unreachable from the internet **inbound**.

* Not requited for IPv6, since they're **inherently routable** on the internet ( IPv6 have **globally unique**  provide a unique address for each device).
* **Outbound Internet Access**: Resources in a private subnet route outbound internet requests to either the NAT Instance or NAT Gateway in a public subnet. The NAT then forwards these requests to the internet, using its public IP address.

Public Subnets have the route to the internet gateway, no NAT  is required between the instance and the Internet Gateway

**NACL** ⇒<mark style="background-color:yellow;">stateless</mark>, layer of security for your VPC that acts as a firewall for controlling traffic for inbound / outbound subnet traffic.

**Public Subnet:**

1. **EC2 Instance** → **Internet Gateway** → **Internet**

**Private Subnet:**

1. **EC2 Instance** → **NAT Gateway** → **Internet Gateway** → **Internet** :spider\_web:

<details>

<summary>IP addresses</summary>

Nearly all resources that you launch in your virtual private cloud (VPC) provide you with an IP address for connectivity. The vast majority of resources in your VPC use private IPv4 addresses.

Public IPv4 addresses have the following types:

* **Elastic IP addresses (EIPs)**: Static, public IPv4 addresses provided by Amazon that you can associate with an EC2 instance, elastic network interface, or AWS resource to achieve persistence.
* **EC2 public IPv4 addresses**: Public IPv4 addresses assigned to an EC2 instance by Amazon (if the EC2 instance is launched into a default subnet or if the instance is launched into a subnet that’s been configured to automatically assign a public IPv4 address).
*   Every **IPv6** is AWS is public there is no private range.

    &#x20;**IPv6 addresses are inherently routable** on the internet ( this means that IPv6 has a **globally unique IP address).**  IPv6 access in a private subnet typically involves using **egress-only internet gateways.**

- **BYO IPv4 addresses**: Public IPv4 addresses in the IPv4 address range that you’ve brought to AWS &#x20;

All public IPv4 on AWS are $0.005 per hour ( including Elastic IP)

</details>

## VPC Flow Logs

Captures all IP traffic going into your interfaces

* VPC flow Log
* Subnet Flow Log
* Elastic tech network log

#### What does it do?

* Helps to monitor & troubleshoot connectivity issues
  * Subnets (can’t connect)to internet
  * Subnets (can’t connect)to subnet
  * Internet (can’t connect)to subnet
* Captures **network info from AWS managed interfaces** : ELB, RDS, Aurora
* VPC flow logs data can go to S3, CloudWatch Logs, Kinesis Data Firehose

## VPC Peering

Connect to VPCs with non overlapping CIDR (IP) ranges to route traffic between them.

* Traffic remains in the private IP address space.
* Peering is **not** **transitive** ( must be established for each VPC that needs to communicate with one another).
* Inter-Region VPC peering provides a simple and cost-effective way <mark style="background-color:yellow;">to share resources between Regions or replicate data for geographic redundancy.</mark>

## VPC Endpoints

#### There are two types of VPC endpoints. These allow private connectivity to AWS services without requiring traffic to traverse the public internet.

### **Interface endpoints**

**Mechanism**: Uses **Elastic Network Interfaces (ENIs)** within your VPC. An _elastic network interface_ is a logical networking component in a VPC that represents a virtual network card.

* They can exist with or without PrivateLink.
  * Using ENIs themselves is free, but if they are part of a PrivateLink setup, you incur costs for PrivateLink usage.
* Supports fine-grained security with **VPC security groups**
* You can enable VPC flow log on your ENI to capture information about the traffic going to and from a network interface
* Most instances support <mark style="background-color:yellow;">1 ENI</mark>
  * Smaller instance types like **t3.micro** support only the primary ENI.
  * Larger instances like c5.4xlarge can support multiple ENIs (e.g., 2-8, depending on the instance size)
* You need **subnet and** can't move a ENI to another subnet after it's created. You must attach a ENI to an instance in the same AZ
* You can't detach a **primary network interface from an instance.**
* &#x20;VPC that provide private connectivity to supported AWS services, third-party services, or your own services. Most AWS services (e.g., **Amazon S3**, **Amazon DynamoDB**, **CloudWatch**, **API Gateway**, etc.)

<details>

<summary>ENI - for preserving network configurations <span data-gb-custom-inline data-tag="emoji" data-code="1f9b8">🦸</span> -> attach to instance failover</summary>

#### **Why Use ENI for Failover?**

* **Preserves Network Identity**:
  * The new instance takes over the same IP addresses and Elastic IP (if assigned), so no DNS updates or client-side changes are required.
* **Consistency in Security Rules**:
  * Security group and firewall rules attached to the ENI remain the same, reducing the risk of misconfiguration when a new instance is deployed.
* **Rapid Recovery**:
  * You don't need to recreate the networking configuration—just attach the existing ENI to the new instance.
*   **Support for Stateful Services**:

    * For services relying on a specific IP address or Elastic IP for identification or communication, the ENI ensures continuity.



    If your EC2 instance fails, you can detach the ENI (which already has your network settings like IP addresses, security group rules, and permissions) and attach it to a new instance. This means:

    * **No need to reconfigure network rules** (e.g., allowing TCP but not SMTP).
    * The new instance gets the same IP address and network setup as the failed one.
    * Your service resumes quickly with minimal disruption.
    * We can move traffice from one instanceto another
    * **Bound to AZ**

</details>



### **Gateway endpoints (the traffic within the AWS network)**

**Mechanism**: Creates an entry in the VPC route table that directs traffic to the service. To allow instances in **private subnets** (which don't have internet access) to interact with **S3** or **DynamoDB** securely.

* Make use of targets specific IP routes in an Amazon VPC route table, in the form of a prefix-list, used for traffic destined to Amazon DynamoDB or S3.
* Gateway endpoints are **free to use**; you only pay for the data transfer to/from the services
* Limited to **Amazon S3** and **Amazon DynamoDB, gateway endpoint is free**
  * **Use Cases**:
    * Access resources from private subnet to interact with S3 or DynamoDB from within your VPC.
    * Avoiding the use of NAT gateways for accessing S3 or DynamoDB.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>



