---
icon: lock-keyhole
---

# IAM

* **User:** can belong to multiple group or don't have to belong to a group(not best practice).
* IAM User represents the person or service that  interacts with AWS.
* **Groups:** can only contain users not other groups.
* **Roles:** for EC2, Lambda etc, or applications in services
* **Policies:** (identity (manages; inline) and resource based (trust))→ JSON format document that outlines permissions for groups, users.
* **IAM Credentials report:** lists all account's users and the status of their credentials.&#x20;
* **IAM Access Advisor:** shows the service permissions granted to a user and when/which services were last accessed.
* **Best practice:** use temporary security credentials (for IAM roles) instead of generated **access keys.**
* **Best practice:** for root and IAM users to use both password + MFA, rotate access keys.
  * AWS provides guidelines for access key rotation every 90 days.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## IAM Policies ( Group, Inline)

* Policies can be attached to IAM users, groups, or roles. Give broad permissions.
* Groups <mark style="color:red;">can’t be nested</mark>, can contain users not groups.
* Granular <mark style="color:red;">least-privilege</mark>. Never give more permissions than needed.
* Users inherit policies from groups they're in.
* AWS evaluates <mark style="color:red;">permissions based on the most restrictive policy</mark>. If the IAM policy allows access but the bucket policy denies it, the user will be denied access.
* A "deny by default" model for permission:

```mermaid
graph LR
  A[Decision starts at 'Deny'] --> B([Evaluate All Available Policies])
  B --> C{Explicit Deny? }
  C -->|Yes| D[Final decison=Deny]
  C -->|No| F{Explicit Allow?}
 
 
  F -->|Yes| K[Final decision=Allow]
  F -->|No| E[Final decison=Deny]





```

* Group policies allow for easy control of permissions for all users in that group simultaneously.
* Inline policies - attached directly to a single user. Good practice for Highly Specific Access or Temp Access requirements.
* Define permissions for Users. Can define specific to service -> Actions -> resources.

#### Achieving Fine Grained /Layered security

Use IAM and bucket policies. IAM policy for DevOps group can grant access to the S3 bucket action (`"Action": "s3:GetObject",`) but Bucket policy (_also a JSON documents specifies permissions at the bucket level_) can enforce additional restrictions on specific objects / certain documents.

## MFA&#x20;

MFA -> password + a security device&#x20;

* Virtual MFA device ( Google Authenticator; Authy)
* Physical key, like YubiKey (U2F)
* Hardware key fob MFA device for AWS GovCloud(US)
* Hardware key fob

## IAM Roles ( for services, apps, user)

> Common roles include EC2 instance, Lambda functions

Services, apps, user can temporarily assume certain permissions without needing to share long-term credentials like access keys with IAM Roles.

The **Lambda Execution Role** is an **IAM Role** that has both **permissions** (via a Permission Policy) and a **Trust Policy** attached. Lambda **Resource-Policy** controls which **external entities** (other AWS services, accounts, or users) have permission to **invoke** or interact with the Lambda function ( ie **S3 or API Gateway to invoke** it,&#x20;

1.  **Permission policy:** permissions i.e. **what actions** (API calls) app/service can perform.

    \-> You provide IAM Role when you create a function, and Lambda assumes the roels when it needs to read objects from an S3 bucket, the Permission Policy grants `s3:GetObject`

    \-> a Lambda writes to DynamoDB&#x20;
2.  **Trust Policy:**  specifies who (or what) is allowed to assume the role.

    \-> For AWS Lambda, the **service principal** is `"lambda.amazonaws.com"`, which is allowed to take the action `"sts:AssumeRole"`. This enables Lambda to assume the role and execute the function on behalf of the service, while gaining the permissions needed to call the AWS Security Token Service (AWS STS) `AssumeRole` action.



## Access Keys :key: for CLI, SDK (associated with IAM User)

> Are long-term credentials for an IAM **User** or root user. Give programmatic access to the AWS CLI or AWS API (directly or using the AWS SDK).&#x20;
>
> * Generated in management console.
> * Same keys for CLI and any SDK.
> * Users generate and  manage their own keys.
> * Applications / scripts use the access key pair to sign requests to AWS services.

&#x20;**Access keys have two parts:**

1. Access key ID like username (for example: `AKIAIOSTUTORIALSDOJO`)
2. Secret access key like password (for example: `wJalrXUtnFEMI/K7MDENG/bTutorialsDojoKEY`







#### Useful Links

[https://docs.aws.amazon.com/IAM/latest/UserGuide/reference\_policies\_elements.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference\_policies\_elements.html)
