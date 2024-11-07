---
icon: table
---

# RDS

Managed DB service for SQL, automatically scales, but we don't have ssh access, but get patching, provisioning.

Maximum storage Threshold ( not to scale indefinitely) -> configure is Free storage % is less than 10% of Total storage will auto-scale scale ( 6hrs since last modification ,condition persist over 5min)

## Engines

* PostgreSQL
  * Up to **5 Read replicas**
  * support for complex queries
* MySQL
  * Up to **5 Read replicas**
* MariaDB
  * strong to support MySql application, is a fork of MySQL, used as drop-in replacement
  * Up to **5 Read replicas**
  * General Purpose SSD (gp3) includes 3,000 IOPS at no additional cost independent of volume size.
*   Oracle

    * Up to **5 Read replicas**
    * You can run Amazon RDS for Oracle under two different licensing models – “License Included” and “Bring-Your-Own-License (BYOL)”. The “BYOL” model is designed for customers who prefer to use existing Oracle database licenses or purchase new licenses directly from Oracle.


* Microsof SQL Server
  * **No read replicas, use Availability Groups for high avail**
  * IBM DB2
* Aurora PostgreSQL/MySQL&#x20;



### **Aurora** :woman\_office\_worker:**- AWS-native relational database** compatible with MySQL and PostgreSQL

a proprietary technology from AWS, and is designed to improve upon standard MySQL and PostgreSQL

* Fully managed by RDS, which automates time-consuming administration tasks like hardware provisioning, database setup, patching, and backups.
* HA native
* Aurora allows up to **15** low-latency read **replicas**
  * Has Reader and Writer instance&#x20;
  * You're charged  :dollar: for the replicas used
  * Set up Read Replicas Auto Scaling policy: based on CPU usage or average number of connections
  * Storage is designed to scale across 100s of volumes, providing **automatic failover** and **fault tolerance**. It replicates data across three Availability Zones (AZs), with 6 copies of the data (2 copies in each AZ) :muscle:
  * Only the primary **(master)** instance accepts **writes**, while read replicas handle read traffic. If the primary fails, Aurora promotes a replica to become the new master, **but at any given time, only one instance accepts writes.**
    * <mark style="color:purple;">Aurora provide Writer Endpoint (DNS name) pointing to the master</mark>&#x20;
    * <mark style="color:purple;">Reader Endpoints connect to the Load Balancer</mark>
* Features a distributed, fault-tolerant, **self-healing with peer-to-peer replication** system that auto-scales up to **64TB per database instance.** It delivers high performance and availability with up to 15 low-latency read replicas, point-in-time recovery, continuous backup to Amazon S3, and replication across three Availability Zones (AZs).
* Aurora MySQL automatically replicated across 3 AZs
* Prices 1/10 of the cost of commercial DBs like Oracle or SQL Server
* Cloud optimised and claims **5x faster** performance over standard MySQL on RDS, 3x over Postgres on RDS
* Continuous **backup** to S3, and replication across three AZs.
  * Aurora’s **storage grows automatically** in **10GB increments as needed,** from min 10GB up to a maximum of **128TB**.
  * Costs 20% more than RDS but more efficient
  * **No free tier**



## RDS Deployments

RDS Multi-AZ deployments’ - high availability, not for scaling.

* Typically SYNC replication to standby DB in another AZ&#x20;
* Failover to cross AZ failover replica if main DB crashes, it ensures HA,  possible because there's one DNS name for main and standby DB
  * loss of instance stoarge
  * loss of AZ
  * loss of network
* Data in Failover AZ is passive until RDS will trigger a failover, fully automatic
* DR strategies

&#x20;Multi-Region deployments -  purpose is disaster recovery and local performance.

* Cross region replication costs
* Good performance because they read from local DB

**RDS Read replicas - purpose is scalability for reads.**&#x20;

Purpose to offload read-heavy traffic from the primary database to these replicas, improving the overall read performance.

* replicas for reads, with ASYNC replication => reads are eventually consistent\*
* can be Cross AZ or Cross Region or within AZ
* replicas can be promoted to own DBs
* Ensure it is for SELECT type of queries, ie enable analytics reporting from DB besides main application reads / writes&#x20;
* Exception for cross AZ (same Region) network costs ( as it is a managed RDS service) - no fee

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### Blue /Green deployment

Creates staging env. It copies a production database environment to a separate, synchronized staging environment. It stays in sync with the current production environment using logical replication.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## RDS Proxy&#x20;

allows for apps to pool & share connections to RDS instead of establishing new

* **Reduced database load**: By reusing existing connections, it minimizes the strain on RDS, especially during traffic spikes.
* **Enhanced security**: RDS Proxy enforces **IAM authentication**, simplifying credentials management and reducing the risk of exposing database passwords.
* **Lambda connection management:** lambdas often experience high concurrency and can create many database connections quickly. RDS Proxy **pools and reuses connections**, which prevents Lambda from overwhelming the database with too many simultaneous connections.
* No application code change, without a proxy, your application would need to handle failover logic manually,
* **VPC-Only Access**: RDS Proxy **never exposes** the database to the public internet; it is **restricted to your VPC**, ensuring private and secure access.

## **Security Details for RDS:**

1. **Encrypting an Existing Unencrypted DB**:
   * To encrypt an unencrypted RDS instance, **create a snapshot**, then **restore it as an encrypted instance**.
   * **Important**: If the master instance was not encrypted at creation, any **read replicas** created from it **cannot be encrypted**. To create encrypted replicas, the master instance must first be encrypted.
2. **SSH Access**:
   * RDS instances do **not** provide SSH access, except for **RDS Custom** instances (for database engines like PostgreSQL and MySQL, or SQL Server), which are part of the **RDS Custom for SQL Server** offering.

