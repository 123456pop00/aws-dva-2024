---
icon: binary-circle-check
---

# Encryption

{% hint style="info" %}

{% endhint %}



## KMS - key management service

* Limit for KMS Api call is 4 KB -> larger use Envelope Encryption
* Envelope encryption uses DEK
* **S3 Bucket Key** is used in conjunction with **SSE-KMS** to reduce the overhead of using KMS for encryption of every sing object we put in a bucket, in order to improve performance by reducing the number of **AWS KMS API calls** needed when encrypting objects.&#x20;
  * When you store an object in an S3 bucket, which beed configures with SSE-KMS, S3 bucket key is created automatically. The **S3 Bucket Key** uses only **one KMS call** is made to encrypt the **Bucket Key** itself. So instead of calling KMS each time, S3 uses the **S3 Bucket Key** to encrypt the object. We make API call for every **PUT** request. The **S3 Bucket Key** itself is encrypted with the KMS key and stored in S3, reducing the need for KMS calls for every object.
  * **Without S3 Bucket Key**: Every time you upload an object, S3 encrypts the object and calls the KMS service to encrypt the object’s key.
  * **S3 Bucket Key** is stored in S3 as metadata for that object.
  * While the **S3 Bucket Key** is shared for objects within the same bucket, it is still encrypted with the **CMK**, so the risk is mitigated by the fact that your **CMK**'s permissions are tightly controlled.
  *   :exclamation:if we disable Bucket key with SSE-KMS encryption selected every upload will be an api call to KMS ( when we don't want a shared **Bucket Key** and want each object to be encrypted individually with **KMS**.



### AWS managed keys - free - _aws/service_

<div align="left"><figure><img src="../.gitbook/assets/aws-managed-keys.png" alt="" width="368"><figcaption></figcaption></figure></div>

### Customer Managed (CMK) - _$1/month per key + $0.03 / 10000 API calls to kms_

<figure><img src="../.gitbook/assets/customer-managed-keys.png" alt=""><figcaption></figcaption></figure>

:money\_mouth::dollar::exclamation: **ORIGIN KMS keys costs for the existence**

1. **Key usage**: AWS charges $1 per month for each **Customer Managed Key** (CMK). This is the base charge for having the key available and being able to use it for encryption and decryption operations :scream\_cat:
2. **Request Costs**: In addition to the key charge, AWS also charges for the **requests** to use the KMS key. For example, each encryption and decryption request made using that key (e.g., when encrypting data or accessing encrypted data) has a cost associated with it. The cost for requests is typically **$0.03 per 10,000 requests**.

#### :grey\_question: **When I create an origin KMS key, is it symmetric?**

Yes, when you create a **Customer Managed KMS Key (CMK)** it is **symmetric** by default. A **symmetric key** means that the <mark style="background-color:green;">same key is used for both encryption and decryption operations</mark>. This is the most commonly used key type for most encryption tasks, such as encrypting data stored in S3 or EBS.

KMS also allows you to create **asymmetric keys** (public and private key pairs) <mark style="background-color:yellow;">for use in specific scenarios like digital signatures</mark>, but these need to be explicitly selected during the creation process.



#### :grey\_question: **What’s the default key policy if I don’t specify one?**

If you don’t specify a key policy when creating a **Customer Managed Key (CMK)**, AWS KMS will apply a **default key policy**.&#x20;

* **AWS Account Full Access**: By default, the key policy grants the **AWS account** that created the key full access to the CMK. This means the root user of the account can perform any KMS operation (like encrypt, decrypt, and manage the key).
* **No Access for Other Principals**: Other IAM users or roles in the account will have no access to the key unless you explicitly modify the key policy to grant them access.

Here’s a **sample of the default key policy**:

```json
// IAM policy for KEY 
{
  "Version": "2012-10-17",
  "Id": "default",
  "Statement": [
    {
      "Sid": "Allow administration of the key",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    }
  ]
}
```



* In short: **Root user has full access** 🔑🎉, and **other users** won’t be able to use the key unless you specifically grant them permissions.

**Automatic Key Rotation & Security** 🔄:

* KMS supports automatic key rotation for **symmetric CMKs** 🚀 It’s **automated**, reducing the risks of key exposure over time. By default, AWS rotates keys every year (12 months). This helps meet security standards while simplifying your life. 😅
* 🔑 **Scheduled Key Deletion** 🕒&#x20;
  * after a certain waiting period (7 to 30 days). 📅&#x20;
  * ⏳ **AWS Deletion Rule**: KMS requires a **minimum of 7 days** and a **maximum of 30 days** before a key is permanently deleted after scheduling.
  * if you change your mind during the waiting period, you can **cancel** the deletion
  * **compliance and key lifecycle**: Scheduled key deletion is useful for organizations that have strict key management and data retention policies
  * :warning: <mark style="background-color:yellow;">Deleting a key makes all data encrypted under that key unrecoverable</mark>
  * :warning: <mark style="background-color:yellow;">Keys scheduled for deletion cannot be used with lambda etc</mark>

🏷️ **Aliases for KMS Keys** 🤖

Instead of using a long string like:\
`arn:aws:kms:region:account-id:key/key-id`,\
You can use an alias like:\
`alias/myAppEncryptionKey`

## APIs

1️⃣ **Encrypt**

**Input**:

* **Plaintext data** (≤ 4KB).
* **CMK ARN** or alias for the key to encrypt with.

**KMS Action**:

* KMS uses the specified **CMK** to encrypt the plaintext.
* It applies envelope encryption if needed.

**Output**:

*   **Ciphertext Blob** (encrypted version of your data is returned the in the API response.



    **Save the ciphertext**:

    *   Write it to a file:

        ```arduino
        fs.writeFileSync("encrypted-file.bin", ciphertext.CiphertextBlob);
        ```
    *   Or upload it to an S3 bucket:

        ```php
        await s3.putObject({
            Bucket: "my-bucket",
            Key: "encrypted-file.bin",
            Body: ciphertext.CiphertextBlob
        }).promise();
        ```

2️⃣ **Decrypt**&#x20;

**Input**:

* **Ciphertext Blob** (encrypted data).
* Optional: **Encryption Context** (if used during encryption).

**KMS Action**:

* KMS determines the CMK used to encrypt the data.
* It verifies permissions and decrypts the ciphertext.

**Output**:

* Plaintext Data&#x20;

```mathematica
codeCiphertext Blob + Encryption Context ➡️ Decrypt API ➡️ Plaintext Data  
```

#### 3️⃣ **Re-Encrypt**

**Input**:

* **Ciphertext Blob** (encrypted data).
* **New CMK ARN or alias** (the key you want to re-encrypt with).

**KMS Action**:

* KMS decrypts the ciphertext using the original CMK.
* Immediately encrypts it again using the new CMK.

**Output**:

* **New Ciphertext Blob** (encrypted with the new CMK).

```sql
Ciphertext Blob + New CMK ➡️ Re-Encrypt API ➡️ New Ciphertext Blob  
```

## DEK

* The **Data Encryption Key (DEK)** is a practical workaround for the 4KB encryption limit of KMS APIs since `Encrypt`, `Decrypt` can only handle data up to **4KB** directly.

#### 🛠 APIs:

1️⃣ **GenerateDataKey API**:

* You get **two things**:
  * **Plaintext DEK** (used for encrypting your data).
  * **Encrypted DEK** (encrypted using your CMK, stays safe for later use).
* The **CMK** (Customer Master Key) stays securely in KMS—it’s used to encrypt and decrypt the DEK, not the data itself.

```bash
#Call GenerateDataKey
   ⬇️
2. KMS returns:
   - Plaintext DEK (you use this to encrypt your file)
   - Encrypted DEK (you store this for future decryption)
   ⬇️
3. You encrypt your file locally with the plaintext DEK.
```

2️⃣ **GenerateDataKeyWithoutPlaintext API**:

* You only get the **Encrypted DEK**.
* No plaintext DEK is returned, so you can’t encrypt new data directly with it—only useful if you already have encrypted data needing decryption later.

**🔑 Summary:**

* Use `GenerateDataKey` if you need to encrypt or decrypt data locally.
* Use `GenerateDataKeyWithoutPlaintext` if you only need to store an encrypted DEK for later.

#### Useful likns

* [https://docs.aws.amazon.com/cli/latest/userguide/cli\_kms\_code\_examples.html](https://docs.aws.amazon.com/cli/latest/userguide/cli_kms_code_examples.html)
* [https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/introduction.html](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/introduction.html)
