# EC2 Storage

AWS Cloud native storage options

* Block storage - EBS, instance store
* File storage - EFS
* Object - S3 + tiers (Glacier etc)

## Block

_means data is stored in fixed-size block or like a stack of building blocks where each block can be individually accessed, modified, or deleted, is designed for workloads that require persistent storage accessible by EC2 instances._

:heavy\_plus\_sign: When a file is updated, the whole series of blocks aren't all overwritten this allows for fast consistent read/write operations and low latency.

:heavy\_minus\_sign: Access pattern is random, and requires managing and organizing blocks, which can be more complex than object storage

### EBS&#x20;

* **Persists independently** from the life of its associated instance - data persists after instance termination, can be used to re-create instance and mount the same EBS volume
* Attach / detach with persistence is useful for failover, or attach on demand
* To **attach additional volume**  ensure AZ is the same like eu-noth-1a eu-noth-1b
* **Bound** to specific **AZ** but if you **snapshot** it, you can move across AZ or Region. Snapshots are stored in S3
  * Recycle Bin for EBS Snapshots allows to retain deleted snapshots for a specified duration (from 1 day to 1 year), provides protection against accidental deletion, ie restore in minutes
  * EBS Snapshot Archive tier is 75% cheaper, but takes up to 24-72 hr restore time
  * **Snapshots are crash-consistent,** they capture all data written to the disk up to the moment the snapshot starts, but anything in memory (not yet written to disk) is lost
  * It's possible to take EBS snapshot while the **volume is still attached and in use,** but recommended approach for full data integrity, is to detach volume, or pause critical application, or "**flush memory**" i.e. force any unsaved data in memory to write to the disk before taking a backup
  * **Basic DR Strategy** involves creating **cross-regions** backups (like EBS snapshots) in a different region to recover from larger outages or disasters, same AZ snapshots are typically cheaper as they don’t incur cross-region transfer fees. Both AZs within the same region share the same pricing for EBS snapshots
  * **EBS Snapshot Policy -** automates the creation and deletion of snapshots, where you set rules for frequency, retention, and target volumes
  *   **Incremental backups,** i.e. for subsequent backups, only the blocks of data that have changed since the most recent snapshot are saved

      <div align="left">

      <figure><img src="../../.gitbook/assets/Screenshot 2024-11-04 at 13.19.04.png" alt="" width="375"><figcaption></figcaption></figure>

      </div>
  * create point-in-time backups through EBS snapshots
  * **Fast Snapshot Restore (FSR) -** reduces latency for restoring snapshots to volumes in specific AZs, making new volumes available almost immediately
* Provision capacity in advance ( GBs, IOPS)
* Free Tier : 30GB of free EBS on SSD or HDD
* Through network attached  'drive' or 'network usb'  to the instance
* It uses network ( _=read latency)_ communication
* Delete on termination by default is enabled for **root volumes**, not for additional volumes

#### EBS Multi-Attach&#x20;

_For example, for a) High-Availability Clustered Applications, this allows the cluster to read and write to shared storage, ensuring high availability and data consistency even if one instance fails; b) distributed file systems, where multiple instance require parallel access, c) when an ultra-low latency, consistent high IOPS, or block-level storage are necessary_

* only available via io1 / io2 type volume
* limited to 16 EC2 instances
* attach same EBS volume to multiple instances **in the same AZ,** with each instance having **full rw permissions** to the volume
* must be cluster-aware file system

**Use case:**&#x20;

* To achieve higher application availability
* To achieve concurrent writes

#### **6 Volume Types**

<div align="left">

<figure><img src="../../.gitbook/assets/Screenshot 2024-11-04 at 12.01.28.png" alt="" width="375"><figcaption></figcaption></figure>

</div>

<details>

<summary>SSD vs HDD</summary>

Volumes using _solid-state drive (SSD)_ and the older spinning _hard drives (HDDs)_.

* **SSD**: Faster, electronic, ideal for quick access and performance-sensitive applications.
  * Good for high-performance workloads, random access, OLTP databases
  * **Small, Frequent Operations**: Ideal for applications requiring many small, frequent read/write operations, such as databases and transactional workloads.
  * **Speed**: Much faster access times because there are no moving parts; data retrieval is instantaneous.
* **HDD**: Slower, mechanical, good for large data storage with sequential access&#x20;
  * Good for large, infrequently accessed data.
  * **Speed**: Slower access times due to mechanical parts (spinning disks and moving read/write heads).

<img src="../../.gitbook/assets/Screenshot 2024-11-04 at 14.10.53.png" alt="" data-size="original">



</details>

1. **General Purpose SSD (gp3)**: Balanced price and performance, suitable for a wide range of workloads.
   1. General Purpose SSD latest generation **gp3** baseline performance scales linearly at 3 IOPS per GiB with a minimum of 100 IOPS, burstable to 3000 IOPS. Max **16,000 IOPS and throughput 1000 Mib/s**
      1. **Independent Configuration**: IOPS and throughput can be configured separate from the volume size, allowing for more flexible performance scaling. <mark style="color:purple;">**Independently provision IOPS for gp3**</mark> :sunflower:
   2. General Purpose SSD (gp2) have **linked IOPS & throughput**, providing **3 IOPS per GiB** with a maximum of **16,000 IOPS**.
   3. System **boot volumes**
   4. <mark style="color:purple;">**gp2**</mark> does <mark style="color:purple;">**not support independent IOPS provisioning;**</mark> its IOPS scales automatically
   5. Recommended for most workloads, test, dev env
   6. Interactive, low-latency
2. **Provisioned IOPS SSD (io1, io2)**: High-performance storage for I/O-intensive applications, offering consistent and low-latency performance, high-throughput.
   1. Ideal for **mission** **critical business applications**, high-performance SSD (io2) for in-memory data processing and real-time analytics, _sap hana, oracle, mongodb_
   2. For applications requiring over 16,000 IOPS, like DB requiring performance and consistency
   3. System **boot volumes**
   4. <mark style="color:purple;">**Independently provision IOPS from storage size for io1 , io2**</mark>:sunflower:
   5. Provisioned IOPS SSD io2 **have max PIOPS 256,000, sub-millisecond latency**
      1. **support EBS multi-attach**
   6.  Provisioned IOPS SSD (io1) volumes support between 100 and <mark style="color:red;">**64,000**</mark> IOPS **depending on the volume size**. For io1 volumes, you can provision up to <mark style="color:red;">50 IOPS per GiB.</mark>&#x20;

       * SSD (io1) -> Size = 4 GiB, IOPS = Min: 100 IOPS, Max: 200 IOPS
       * SSD (io1) -> Size = 500 Gib, IOPS = 3000


3. **Throughput Optimized HDD (st1)**: Low-cost HDD for frequently accessed, <mark style="color:red;">throughput-intensive</mark> workloads like log processing, ETL workloads, big data, data warehouses. **Higher latency vs SSD**
   1. Offer lower IOPS compared to SSD volumes, **focusing** **on throughput** rather than random I/O operations
   2. up to 16TiB
   3. max throughput of 500 Mib/s - max IOPS 500
   4. st1 volumes are a lower-cost option for workloads that require high data transfer rates but do not need the low-latency performance of SSDs
   5. <mark style="color:red;">**cannot be used as boot volumes**</mark> for EC2 instances. Boot volumes must be of type **SSD (gp2 or gp3)** or **provisioned io1/io2.**
4. **Cold HDD (sc1)**: Lowest-cost HDD designed for less frequently accessed data, suitable for cold storage and large data sets archival purposes
   1. max throughput is 250 Mib/s - max IOPS 250
   2. for lowest cost possible&#x20;

### Instance Store - ephemeral block-level storage

* volume is attached to the **underlying physical host**, if <mark style="color:red;">**you stop, hibernate, or terminate an instance, any data on instance store volumes is lost.**</mark> The reason for this, is that if you start your instance from a stop state, it's likely that EC2 instance will start up on another host. A host where that volume does not exist. Instances are just VMs, so underlying host can change between stopping and starting an instance
  * every block of the instance store volume is cryptographically \* erased
  * data persists if <mark style="color:red;">**rebooted**</mark>
* Inside the instance store data is stored on block devices, number of block devices depends on instance type & size (m,r,c)
* Copy data to EFS, S3 or EBS if you want to retain data after instance is deleted
* Very high IOPS up to 3.3million on i3.16xlarge or i3.metal instance sizes
* Good for **buffer** (data that's being moved from one place to another, video streaming service buffering video data to ensure smooth playback), **scratch data** (for intermediate data processing, worked on, like image editing), **cache** (frequently accessed data, to speed up future access), any other temp content

## File

### EFS

* Managed NFS( network file system) that can be mounted on 100s of EC2
* EBS Multi-Attach like multithreading with data
  * lock mechanism
  * version controls
  * last write wins
  * merge changes
* POSIX file system (\~Linux) that has a standard file API
* Works with Linux EC2s in multi AZ, designed for multiple EC2 instances to access simultaneously4
* Not compatible with Windows AMIs

#### Features

**Elastic -** pay only for capacity used, scales with capacity, <mark style="color:red;">supports workload spikes</mark> &#x20;

**Serverless & scalable -** no provisioning, scale capacity, connections, and IOPSs, grows to petabyte-scale NFS automatically

* Ancestery.com case ( i.e., multiple scientists to perform genomics research and need to manage unpredictable workload so EFS scales compute and storage resources up or down, not pay for idle resources

**Performant -** 10s GB/s and 500,000+ IOPS. Performance settings are customisable upon setup.

<figure><img src="../../.gitbook/assets/Screenshot 2024-11-05 at 10.55.15.png" alt=""><figcaption></figcaption></figure>



**3-AZ durable and all-AZ available** - designed for 11 9s of durability and 99,99% availability SLA

* For sandbox, backups, dev, pre-prod workloads use One Zone (single AZ), also compatible with IA

**Concurrent access for 10,000s of connections -** EC2 instances, lambda invocations, containers

**Persistent storage type** -  if Wordpress traffic goes up, you want more servers, but you need access to the same static content

**Two storage classes -** lifecycle based cost optimisation, need to implement lifecycle policy.

* Standard Tier (_on first access in IA, Archive move back to Standard class_)
* EFS-IA (_after n-days move to IA, i.e. 30 days since last access, retrieval cost but lower storage costs_)
* Archive (_rarely accessed files, 50% lower costs_)

#### EFS vs EBS



### Object







