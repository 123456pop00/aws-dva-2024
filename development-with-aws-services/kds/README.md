---
icon: arrow-progress
---

# KDS

> Purpose of KDS is capture & stream when you need immediate results with integration of MAF with  instant 'put-to-get' \~delay 1 second for data consumers.
>
> * Real-time analysis
> * Powering event-driven architectures
> * Log processing

* Capture data with KDS or MSK and send to Managed Apache Flink for complex real-time analytics.



## Dual-Write problem for distributed systems

Goal is to stream **reliable events in KDS**&#x20;

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

* **ECS Service** receives HTTP requests, processes the data, and stores it in **DynamoDB**.
  * If instance cannot process data, it sends it to the **SQS -> SendMessage API** to **DLQ**.&#x20;
* **DynamoDB** stores the data and emits changes to **DynamoDB Streams**. <mark style="color:blue;">Address Dual-Write to down-stream</mark>
* **DynamoDB Streams** breaks down the data into shards, which enables parallel processing of data changes in the streams.
  * The streams give you **eventual consistency** on updates to your DynamoDB table. By subscribing to these streams, you can track any **insert**, **update**, or **delete** real-time.
  * **Retention and Replay**: DynamoDB Streams retains the change records for **24 hours**, allowing consumers to replay and process data within that window.
* **ECS Service** ('<mark style="color:blue;">compensation layer</mark>') processes the data from **DynamoDB Streams and Apply any transformations for downstream consumer services.**
  * ECS services **reconcile** the state of the data and apply logic to ensure it’s in sync.
  * If an event cannot be processed correctly (e.g., an error occurs), the message is sent to an SQS queue  configured with a DLQ.
* &#x20;**SQS DLQ** stores these failed messages for later investigation or reprocessing.
* **Lambda Handles Failed Events from SQS / DLQ**
  * Lambda can retry failed events from SQS/DLQ as Lambda functions can be used to process events asynchronously.

