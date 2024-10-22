---
description: >-
  Most AWS services are region-scoped. Each region has minimum 3 AZ and maximum
  6. Each AZ has one or more discrete data centers.
---

# Global Infrastrucutre

* Regions (34) -> AWS has largest infrastructure footprint of any cloud provider (31% market share).
* Availability Zones (108) -> Deploying in at least 2 AZs is a best practice to ensure redundancy / failover. AZs are located far from each other to ensure disaster recovery and ensures that issues in one AZ do not affect others.  AZ consist of one or multiple physical data centers that have isolated power, networking.
* Edge locations / PoP (600+) ->  'like :pizza: delivery :truck:' don't run applications but cache common requests. Ex: CloudFront CDN uses to store copies of content closed to end-user for high availa­bility.
* Local Zones -> 'like a :pizza: shop'  to support low latency sensitive applications in geographic proximity to users, core purpose is to achieve single-digit millisecond latencies. We extend  VPC to more locations, with local zone subnet, for example, we have N.virginia region with 3 AZs and extend it by defining a Local Zone.&#x20;

<!---->

* [x] Not full-fledged DC, they replicate the full infrastructure of an AZ but rather provide a subset of services ( i.e., compute / storage/ data base) to support applications that require latency access. _Ex: gaming, video conferencing/streaming (we setup encoding and transcoding services in a Local Zone) , real-time analytics, IoT applications that process data from devices in real-time we use LZ for the data ingestion and processing logic to reduce latency._
* [x] Local Zones don't have  additional fees. You pay only for the services you consume in Local Zones.
* [x] Users have full access to services in the **parent** region, and we  have the option of partitioning workloads between a Local Zone and a Region to increase availability.

<figure><img src="../.gitbook/assets/Screenshot 2024-10-22 at 12.08.57.png" alt=""><figcaption></figcaption></figure>

#### **What to put in a LZ ?**&#x20;

* Workloads with unpredictable traffic spikes might benefit from the flexibility of Local Zones to quickly provision resources.
* Do we have high throughput reqs?
* Always evaluate FR, NRFs ( performance, cost) and workloads together, suitability. Workload characteristics will impact the design :heavy\_check\_mark:Stateless applications, which are often _easier_ to run in Local Zones. Stateful applications may require additional considerations for data consistency ( :moneybag: :arrow\_up:) and persistence. :exclamation:Do we need to providing real-time availability over immediate consistency? ie. Overall feed state not completely consistent at any given moment -> we aim to for '_eventual consistency'_ as we propagate over time asynchronously changes may be made on one server (or node) and it takes time to replicate through the system to other nodes.
* Always management overhead of running LZs + cost of running services in exatra LZs.
* Running LZs on top of AZs always introduce  🧢 CAP -> Consistency, Availability, Partition Tolerance risks. EX.: we can accept replication latency,  ie delays in propagating changes to other nodes.

:point\_right: **Design for eventual consistency.**

:point\_right: **Data replication strategies.**

:point\_right: **Last write wins (LWW) conflict mechanisms, versioning.**

:point\_right: **Consider** distributed databases designed to handle partition tolerance and provide eventual consistency, such as **DynamoDB**, **Amazon Aurora with Global Databases**.

<details>

<summary><mark style="color:blue;">Considerations for a service location</mark></summary>

1. **Compliance & regulations** -> data residency laws (ISO27001, GDPR)
2. **Price** -> pricing vary by region (ex: ie Brazil tax system makes the same payload more expensive than US region)
3. **Availability within the region** -> not all features are available globally
4. **Proximity**  -> reduced latency

</details>

## Global  Services

* **IAM** → lets you securely control access to AWS services and resources.
* **Route 53** → is a scalable and highly available Domain Name System (DNS) and Domain Name Regist­ration service.
* **CloudFront** (Content delivery network → provides a way to distribute content to end users with low latency and high data transfer speeds.)
* **WAF** → protects web applic­ations from attack by providing web traffic filtering against common web exploits like SQL injection.

## Region Scoped

* EC2 (IaaS)
* RDS
* DynamoDB ( global tables allow replication across regions :star:)
* Lambda
* Rekognition

### Examples

1. Multi-AZ RDS setup \[Add rds mermaid example]



