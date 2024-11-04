# EC2 Storage

AWS Cloud native storage options

* Block storage - EBS, instance store
* File storage - EFS
* Object - S3 + tiers (Glacier etc)

### Block

_means data is stored in fixed-size block or like a stack of building blocks where each block can be individually accessed, modified, or deleted, is designed for workloads that require persistent storage accessible by EC2 instances._

:heavy\_plus\_sign: When a file is updated, the whole series of blocks aren't all overwritten this allows for fast consistent read/write operations and low latency.

:heavy\_minus\_sign: Access pattern is random, and requires managing and organizing blocks, which can be more complex than object storage

### EBS&#x20;

* **Persists independently** from the life of its associated instance - data persists after instance termination, can be used to re-create instance and mount the same EBS volume
* Attach / detach with persistence is useful for failover, or attach on demand
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

### **4 Volume Types**

<div align="left">

<figure><img src="../../.gitbook/assets/Screenshot 2024-11-04 at 12.01.28.png" alt="" width="375"><figcaption></figcaption></figure>

</div>



<details>

<summary>SSD vs HDD</summary>

Volumes using _solid-state drive (SSD)_ and the older spinning _hard drives (HDDs)_.

* **SSD**: Faster, electronic, ideal for quick access and performance-sensitive applications.
  * Good for high-performance workloads
  * **Small, Frequent Operations**: Ideal for applications requiring many small, frequent read/write operations, such as databases and transactional workloads.
  * **Speed**: Much faster access times because there are no moving parts; data retrieval is instantaneous.
* **HDD**: Slower, mechanical, good for large data storage.
  * Good for large, infrequently accessed data.
  * **Speed**: Slower access times due to mechanical parts (spinning disks and moving read/write heads).

<img src="../../.gitbook/assets/Screenshot 2024-11-04 at 14.10.53.png" alt="" data-size="original">



</details>



1. **General Purpose SSD (gp3)**: Balanced price and performance, suitable for a wide range of workloads.
   1. General Purpose SSD (gp3, gp2) baseline performance scales linearly at 3 IOPS per GiB with a minimum of 100 IOPS, burstable to 3000 IOPS. Max **16,000 IOPS**.
   2. System boot volumes
   3. Recommended for most workloads
   4. interactive, low-latency
2. **Provisioned IOPS SSD (io1,io2)**: High-performance storage for I/O-intensive applications, offering consistent and low-latency performance.
   1. ideal for **critical business applicaitons**, high-performance SSD (io2) for in-memory data processing and real-time analytics, _sap hana, oracle, mongodb_
   2.  Provisioned IOPS SSD (io1) volumes support between 100 and 64,000 IOPS **depending on the volume size**. For io1 volumes, you can provision up to <mark style="color:red;">50 IOPS per GiB.</mark>&#x20;

       * SSD (io1) -> Size = 4 GiB, IOPS = Min: 100 IOPS, Max: 200 IOPS
       * SSD (io1) -> Size = 500 Gib, IOPS = 3000


3. **Throughput Optimized HDD (st1)**: Low-cost HDD for frequently accessed, throughput-intensive workloads like log processing, ETL workloads, big data, data warehouses.
   1. Offer lower IOPS compared to SSD volumes, **focusing** **on throughput** rather than random I/O operations
   2. st1 volumes are a lower-cost option for workloads that require high data transfer rates but do not need the low-latency performance of SSDs
   3. **cannot be used as boot volumes** for EC2 instances. Boot volumes must be of type **SSD (gp2 or gp3)** or **provisioned io1/io2.**
4. **Cold HDD (sc1)**: Lowest-cost HDD designed for less frequently accessed data, suitable for cold storage and large data sets archival purposes

### File

### Object







