---
icon: database
---

# DynamoDB

## Key Features :key2:&#x20;

* Fully managed, replicated across AZs, **distributed**(=horizontal scaling)NoSQL DB that can scale both reads and writes horizontally.&#x20;
  * **Strong Schema Flexibility:**
    * While DynamoDB is schema-less, it is  more like schema-**flexible**. Each item in a table can have a different set of attributes, but the **Primary Key is mandatory for every item.**
* Max item size 400KB
* Tables - Standard and Standard IA
* Attempts to manage hot partitions using **adaptive capacity**, which reallocates unused capacity from underutilised partitions to those experiencing high demand. Has limits.
* On-demand and provisioned capacity modes. Switch modes once in 24 hrs. On-demand is :money\_mouth: \*2,5 more expensive no need to plan RCU and WCUs insted you get RRU and WRU (write request units). Best mode for unknown workloads, unpedictable traffic.



## Primary Keys -> mandatory

#### HASH strategy: Primary key = Partition Key

*   High cardinality (many unique values) spreads the workload evenly across partitions.

    * Hot partitions -> too many requests hitting the same partition -> use **DAX in memory DB that** caches the most frequently used data,  offloading the heavy reads on hot keys - only for **read-heavy hot partitions**

    &#x20;**Mitigation Strategies:**

    * **Distributed Keys**
      * **Composite Keys:** Add randomness or a prefix to the Partition Key for better distribution (e.g., sharding `ProductID` by adding a random suffix like `ProductID_1`, `ProductID_2`).
      * **Time-based Prefixing:** If your workload is time-dependent, prefixing with date or hour chunks can distribute writes (e.g., `2024-11-27_ProductID`
    * **Exponential backoff when exception is already encoutered**
    * **DAX for offloading reads**

#### HASH + RANGE strategy: Primary key = Partition Key + Sort Key

* Unique Combination Requirement&#x20;
  * For example, gaming logs, where Partition Key like `UserID` and a Sort Key is `Timestamp` is a common pattern.&#x20;

## Partitions -> logical division

* Each partition can store up to **ten gigabytes (GB)** of data and has a slice of your **read/write capacity units** (RCUs/WCUs).
* All 3 replicas act as **READ replicas**, meaning they can serve read requests.
* Only 1 logically **primary WRITE replica** to process write operations.
  * For **write capacity (WCUs)**:\
    One partition supports up to <mark style="color:red;">1000</mark> **write capacity units (WCUs)**.
  * For **read capacity (RCUs)**:\
    One partition supports up to <mark style="color:red;">3000</mark> **strongly consistent RCUs** or **six thousand eventually consistent RCUs**.
* Optimistic Locking:  concurrency writes/model can be implemented using DynamoDB **conditional writes.**
  * **Optimistic locking** assumes multiple transactions will not conflict with each other. Usually takes note of a version number or timestamp to be checked.  _"I’ll try to do my work, and only if there’s a conflict, I’ll stop and handle it."_
  * **Pessimistic locking** assumes multiple transactions will  conflict  and locks the data while the current transaction is in place. Relational DBs."_I’ll lock the data now to make sure no one else can change it while I’m working on it._
* RCU and WCU are **decoupled** and can be provisioned on demand. Auto-scaling similar to EC2 with min max range.&#x20;
  * RCU and WCUs are spread evenly across all the table's partitions so **Hot Key** (popular item like productId ) or **Hot partitions** or a very heavy items will cause <mark style="color:red;">`ProvisionedThroughputExceededException`</mark>&#x20;
  * Exceeding **burst capacity** will lead to <mark style="color:red;">`ProvisionedThroughputExceededException`</mark>
  * Items over 400KB will lead to  <mark style="color:red;">`ProvisionedThroughputExceededException`</mark>

## Consistency Modes

* <mark style="background-color:yellow;">**Eventually Consistent Reads**</mark>
  * **Fastest**: Reads can be served from **any replica**, typically the nearest.
  * **Tradeoff**: There's a possibility of reading **stale data** because replicas are updated asynchronously.
  * **RCU Consumption**: 1 RCU reads **2 items up to 4 KB each**.
*   <mark style="background-color:red;">**Strongly Consistent Reads**</mark>

    * **Serves only from the Lead (Primary) replica**: Ensures the read is from the latest committed write.
    * **Synchronization**: DynamoDB ensures the data is fully synchronized across replicas before returning results.
    * **Tradeoff**: Higher latency and cost compared to eventually consistent reads.
    * **RCU Consumption**: 1 RCU reads **1 item up to 4 KB** (double the cost of eventually consistent).
    * For API Calls such as  GetItem, BatchGetItem, Scan parameter `ConsistentRead = true`



    **Writes Always Hit the Primary (Lead) Replica**:\
    <mark style="background-color:yellow;">Regardless of consistency mode,</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**all writes**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">go to the lead replica and are</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**synchronously propagated**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">to the other two replicas in the availability zones (AZs).</mark>\
    This ensures durability and eventual consistency across all replicas.

### Strongly vs Eventually Consistent Read :books:

* Each AZ contains **one replica of your data** for durability and high availability.
* Strongly consistent reads are served **only by the&#x20;**<mark style="background-color:yellow;">**LEAD replica**</mark><mark style="background-color:yellow;">.</mark>
* Eventually consistent reads can be served by <mark style="background-color:yellow;">**ANY**</mark>**&#x20;replic**<mark style="background-color:yellow;">**a**</mark>, including the primary.

#### **Strongly Consistent Reads**

* These always go to the **primary replica** (the lead replica).
* This ensures the data is the most up-to-date because the read query checks the version of data written and acknowledged by the primary replica.

#### **Eventually Consistent Reads**

* These can be served from **any replica** (including non-primary replicas).
* Because writes are propagated asynchronously to other replicas, there's a possibility of stale data being read. This is why it's "eventually" consistent. The lead replica doesn't need to be involved in every eventually consistent read

### DB Throttling :fire: :fire\_engine:

**429 (ThrottlingException)**: Automatically retried using exponential backoff and jitter. You have to fix the issue in your application before submitting the request again. So AWS SDKs for DynamoDB automatically retries requests that receive 400 exception.

* If <mark style="color:red;">`ProvisionedThroughputExceededException`</mark>already happened  -> **Use Retry Logic for Throttling**: Leverage the SDK's automatic retries for throttling errors, and **implement backoff strategies to avoid overwhelming the service.**
* Use **DAX** if it is RCU issue



## Reads / Writes

<table><thead><tr><th width="182"></th><th>READS</th><th>WRITES</th></tr></thead><tbody><tr><td><strong>API Calls</strong></td><td><code>GetItem</code>, <code>Query</code>, <code>Scan</code>, <code>BatchGetItem</code></td><td><code>PutItem</code>, <code>UpdateItem</code>, <code>DeleteItem</code>, <code>BatchWriteItem</code> and  for all operations, you must specify the <strong>primary key</strong> (<code>--key</code>) to identify the item being worked on.</td></tr><tr><td><strong>Batch Operations(</strong> for speed &#x26; simplicity)</td><td><strong><code>BatchGetItem</code></strong>: Retrieves multiple items by primary key.<br>Supports up to <strong>100 items or 16 MB</strong> per request.<br>Does not support filters or expressions. Items are retrieved in parallel. </td><td><strong><code>BatchWriteItem</code></strong>: Performs up to <strong>25</strong> <code>PutItem</code> or <code>DeleteItem</code> operations per request or 16 MB.<br>Cannot update items, only create or delete them. Can't do <mark style="color:red;"><code>UpdateItem</code> -> Atomicity Challenge (ie merge attributes, use counter).</mark> Use <strong><code>UpdateItem</code></strong> API for each item in a loop in your application.</td></tr><tr><td><strong>Transactional Mode</strong></td><td>Supports <code>TransactGetItems</code> for ACID-compliant reads.</td><td>Supports <code>TransactWriteItems</code> for ACID-compliant writes (combines multiple write operations).</td></tr><tr><td><strong>Performance</strong> </td><td><p><strong>RCU (Read Capacity Units)</strong>: </p><ul><li>Eventually consistent -> <strong>2</strong>  under 4 KB <strong>items</strong> = 1 RCU; </li><li>Strongly consistent -> <strong>1 item</strong> under 4 KB = 1 RCU.</li><li>Items over 4kb we round up to calculate <span data-gb-custom-inline data-tag="emoji" data-code="1f44d">👍</span><span data-gb-custom-inline data-tag="emoji" data-code="2757">❗</span></li></ul></td><td><strong>WCU (Write Capacity Units)</strong>: 1 write for 1 KB item = 1 WCU.</td></tr><tr><td><strong>Conditional Logic</strong></td><td><p><strong><code>KeyConditionExpression</code></strong> limits the number of items retrieved from the table. Mandatory PK. Used in specific operations like <code>GetItem</code>,</p><p></p><p><strong><code>FilterExpression</code></strong> only narrows down the results after the Query, so it doesn’t save RCUs.</p><p><strong><code>ProjectionExpression</code></strong> minimizes data transferred, saving network bandwidth but doesn’t reduce RCUs either.</p></td><td><p><strong>Atomic Operations</strong>: Conditions ensure operations are atomic, avoiding overwrites or race conditions.</p><p></p><p><strong>Do not replace the need for the primary key</strong></p><p> </p><p><strong><code>ConditionExpression</code></strong>: Common to all conditional writes: </p><ul><li>Existence (<code>attribute_exists</code>, <code>attribute_not_exists</code>)</li><li>Comparisons (<code>&#x3C;</code>, <code>></code>, <code>=</code>, etc.)</li><li>Logical operators (<code>AND</code>, <code>OR</code>, <code>NOT</code>)</li></ul></td></tr><tr><td>Indexes fro Flexibility</td><td>Supports <strong>Global Secondary Index (GSI)</strong> and <strong>Local Secondary Index (LSI)</strong> for querying with alternate key patterns.</td><td>GSIs support eventual consistency; LSIs must be strongly consistent.</td></tr></tbody></table>

### Conditional Expressions

> **Conditional expressions** in DynamoDB are typically specified as part of the parameters in the API request to DynamoDB.
>
> :white\_heart: written in **expression syntax**

Data types of attributes (such as strings, numbers, binary, etc.), and the data type is specified by a single-character code. Here's what the `S` stands for:

* **`S`**: String
* **`N`**: Number
* **`B`**: Binary
* **`SS`**: String Set
* **`NS`**: Number Set
* **`BS`**: Binary Set
* **`M`**: Map (i.e., a nested JSON object)
* **`L`**: List (i.e., an array)
* **`BOOL`**: Boolean
* **`NULL`**: Null

### READS :dark\_sunglasses:

Query a range of users based on their `createdAt` value, you should use a `Query` instead of `GetItem`.

```bash
#GetItem CLI Command
aws dynamodb query \
  --table-name Users \
  --key-condition-expression "createdAt < :date" \
  --expression-attribute-values '{":date": {"S": "2024-11-24T00:00:00Z"}}'

```

Query specific user

```bash
aws dynamodb get-item \
  --table-name Users \
  --key '{"userID": {"S": "12345"}}'

```

#### **KeyConditionExpression ->** conditions that must match **at the key level** (Partition Key and optionally Sort Key) for a Query operation.

* **Mandatory for Query**:
  * Must include the Partition Key (PK).
  * Sort Key (SK) is optional but can have conditions like `BETWEEN`, `BEGINS_WITH`, `=`, `<`, `>`, etc.

```json
codeKeyConditionExpression: "userID = :id AND createdAt BETWEEN :startDate AND :endDate"
```

* Here, `userID` is the PK, and the range condition is applied on the SK `createdAt`.

***

#### **FilterExpression -> a**pplies additional filtering **after the items matching KeyConditionExpression are retrieved**.

* Filters are applied **client-side**, meaning they do not reduce the RCU cost of the Query operation, as all matching items (based on keys) are first fetched from the database.
* **PK not required**: Can filter on any non-key attributes.
* **Not applicable or useful in `GetItem`**, because it return only a **single item.**

```json
--filter-expression "createdAt < :date
```

```bash
#Query all items in the "Tops" category,
# ONLY return items that have a price greater than 500
aws dynamodb query \
  --table-name WorkoutClothesItems \
  --key-condition-expression "category = :category" \
  --filter-expression "price > :price" \
  --expression-attribute-values '{":category": {"S": "Tops"}, ":price": {"N": "500"}}
```

#### **ProjectionExpression -> s**pecifies the attributes to retrieve from the table.

* **Optional**: If not specified, DynamoDB retrieves all attributes of matching items.
* **PK is not mandatory**: You can choose which attributes you want to include in the result set.

```bash
# Return onlyName and Email fields for each user
aws dynamodb scan \
  --table-name Users \
  --projection-expression "Name, Email"

```



### WRITES :pen\_ballpoint: `--condition-expression`

#### Comparison Operators

* **`=`** (Equal to)

`--condition-expression "status = :status"`\
`--expression-attribute-values '{":status": {"S": "active"}}'`

* **`<>`** (Not equal to)

Ensure `balance` is not equal to zero before updating.

`--update-expression "SET balance = :balance"`\
`--condition-expression "balance <> :zero"`\
`--expression-attribute-values '{":balance": {"N": "100"}, ":zero": {"N": "0"}}'`

* **`>=`** (Greater than or equal to)

Check age is greater than or equal to 18 before updating status.

```bash
  --update-expression "SET status = :newStatus" \
  --condition-expression "age >= :minAge" \
  --expression-attribute-values '{":newStatus": {"S": "verified"}, ":minAge": {"N": "18"}}'
```

* **`<`** (Less than)

Only update if lastModified is in specific date

`--update-expression "SET status = :newStatus"`\
`--condition-expression "lastModified < :timestamp"`\
`--expression-attribute-values '{":newStatus": {"S": "inactive"}, ":timestamp": {"N": "1609459200"}}'`

* **`>`** (Greater than)

Ensure `balance` is above  50 before decreasing.

`--update-expression "SET balance = balance + :amount"`\
`--condition-expression "balance <= :maxBalance"`\
`--expression-attribute-values '{":amount": {"N": "20"}, ":maxBalance": {"N": "100"}}'`





#### Existence Operators

&#x20;check if an attribute exists or doesn't exist.

`--condition-expression "attribute_exists(userID)"`

`--condition-expression "attribute_not_exists(userID)"`

The **`attribute_not_exists`** operator can be useful in scenarios where you want to **prevent overwriting** an item or attribute in DynamoDB. This is commonly used to ensure that an attribute is only set if it doesn't already exist.

* **Explanation**: If you are trying to **update** or **put** an item and want to make sure that a certain attribute does not already exist, you can use `attribute_not_exists` as a condition in your **`ConditionExpression`**. This way, if the attribute exists already, the write operation will fail (thus preventing overwriting).

Conditional write: check that `email` attribute **does not already exist** in the table before the `PutItem` operation is allowed to execute.

```bash
# users must have unique email address
aws dynamodb put-item \
    --table-name Users \
    --item '{"UserId": {"S": "user123"}, "email": {"S": "user@example.com"}, "name": {"S": "John Doe"}}' \
    --condition-expression "attribute_not_exists(email)"

```

If someone has already registered with the email, the system returns an error and the registration is put item  fails:

`{ "message": "The conditional request failed" }`



#### Logical

**`NOT`**: Negates a condition

**`AND:`**&#x43;ombines two or more conditions, and the expression evaluates to true only if all conditions are true.

**`OR:`**&#x43;ombines two or more conditions, and the expression evaluates to true if at least one condition is true.

**`BETWEEN:`** Evaluates if an attribute’s value falls within a given range. It works for numbers, strings, and binary types.

* **Syntax**: `attribute_name BETWEEN value1 AND value2`

















## DynamoDB Streams

* Allows to capture a time-ordered sequence of item-level modifications in a table. It's integrated with AWS Lambda so that you create triggers that automatically respond to events in real-time.

