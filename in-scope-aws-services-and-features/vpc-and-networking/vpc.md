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

**NACL** ⇒stateless, layer of security for your VPC that acts as a firewall for controlling traffic for inbound / outbound subnet traffic

**Public Subnet:**

1. **EC2 Instance** → **Internet Gateway** → **Internet**

**Private Subnet:**

1. **EC2 Instance** → **NAT Gateway** → **Internet Gateway** → **Internet** :spider\_web:

<details>

<summary>IP addresses</summary>

Nearly all resources that you launch in your virtual private cloud (VPC) provide you with an IP address for connectivity. The vast majority of resources in your VPC use private IPv4 addresses.

Public IPv4 addresses have the following types:

* **Elastic IP addresses (EIPs)**: Static, public IPv4 addresses provided by Amazon that you can associate with an EC2 instance, elastic network interface, or AWS resource to achieve persistent.
* **EC2 public IPv4 addresses**: Public IPv4 addresses assigned to an EC2 instance by Amazon (if the EC2 instance is launched into a default subnet or if the instance is launched into a subnet that’s been configured to automatically assign a public IPv4 address).
*   Every **IPv6** is AWS is public there is no private range.

    &#x20;**IPv6 addresses are inherently routable** on the internet ( this means that IPv6 has a **globally unique IP address)** Instead, IPv6 access in a private subnet typically involves using **egress-only internet gateways.**

<!---->

* **BYO IPv4 addresses**: Public IPv4 addresses in the IPv4 address range that you’ve brought to AWS using&#x20;

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
* Inter-Region VPC peering provides a simple and cost-effective way to share resources between Regions or replicate data for geographic redundancy.

## VPC Endpoints

#### There are two types of VPC endpoints:

1. **Interface endpoints**
   1. enable connectivity to services over AWS privateLink
   2. These are elastic network interfaces (ENIs) in your VPC that provide private connectivity to supported AWS services, third-party services, or your own services.
2.  **Gateway endpoints (the traffic within the AWS network)**

    1. targets specific IP routes in an Amazon VPC route table, in the form of a prefix-list, used for traffic destined to Amazon DynamoDB or S3

    <figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

VPC Endpoint Gateway: for S3 and DynamoDb



