# KCL

* [x] Maximum number of KCL instances = number of shards, a stream is divided into **shards**, and each shard allows one **KCL instance** to process the records in it. This is because each instance is responsible for processing records from a single shard.
* [ ] Application running KCL needs the IAM permissions

:warning: KCL Maintains Order During Shard Splits and Merges by **Shard Rebalancing.**&#x20;

1. KCL **rebalances** the load between KCL instances. The **instance responsible for Shard A** will continue processing records from the **new shards (A1 and A2)**. If the split results in a new shard, KCL automatically assigns one or more KCL instances to the new shards.
2. KCL maintains a **checkpoint** for the last processed record in a shard.

:warning: **Kinesis Client Library (KCL)** **skips** records if an <mark style="color:red;">**Exception**</mark> is thrown during processing, and the record will not be retried automatically. This behavior is designed **to avoid blocking the entire stream processing flow**, but it can lead to some records being lost in case of failures.

<details>

<summary>Tradeoffs of Split/Merge Elasticity</summary>

The primary tradeoff when using **shard splitting/merging** for **elasticity** is **latency**. The KCL will take time to:

* **Rebalance processing** across instances.
* **Reassign tasks** from one shard to another.
* **Handle any interruptions** caused by the split/merge events.

</details>



#### Default Behavior:

* If a **Kinesis consumer** (using the KCL) encounters an exception while processing a record (for example, if there's an issue with writing data to a database or some other downstream system), the **KCL will skip** that record.
* **No automatic retries** are triggered for those records by default. This means the record will be ignored, and the consumer will continue processing the next record in the stream.

#### Solutions:

* **Use Kinesis Firehose**: if you need  <mark style="color:red;">**automatic retries**</mark> or buffer management, using Firehose can handle retries and data buffering for you.
* **Implement Custom Error Handling**:  catch the exceptions and decide what to do—such as logging the error, sending the record to a DLQ, or attempting a retry.
* **Custom Retry Logic**: If you want retries, you can add custom logic in your application to re-process failed records, such as implementing a delay or retry mechanism, or using a separate queue for failed records (like SQS or DynamoDB).

