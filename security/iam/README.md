---
icon: lock-keyhole
---

# IAM

* **User:** can belong to multiple group or don't have to belong to a group (not best practice). **IAM user** represents the person or service that  interacts with AWS.
* **Groups:** can only contain users not other groups.
* **Roles:** for EC2, Lambda, Cloud9, or other service / application to perform action on the account. **Trust policies** are only associated **IAM Roles**  to specify the trusted entities that can assume the role.
* **Policies:** JSON-format documents that outline permissions for AWS identities (such as groups, users, and roles) and can be divided into two main types: **Identity-based** and **Resource-based** policies.

:exclamation: **Identity-based:** policies attached to users, groups, roles to grant THEM permission to perform actions on AWS resources.&#x20;

Identity-based types:

* **AWS Managed Policies**: These are predefined policies created and maintained by AWS, when you create a Cloud9 role, AWS may automatically attach an AWS managed policy for you.
* **Customer Managed Policies**: These are policies that you create and manage yourself.&#x20;

:exclamation: **Resource-based:** policies attached directly to resource policies like S3 bucket or Lambda function, AWSServiceRoleForAWSCloud9. They define **Trust Entities that can assume the role.**

* **Trust Policies**: This is a specific type of resource-based policy that is attached to an IAM role. It defines the **trust relationships** and specifies which identities (users, services, or accounts) are allowed to assume the role.
* **IAM Credentials report:** lists all account's users and the status of their credentials. Account level report.
* **IAM Access Advisor:** breaks down the permissions granted to a user, role, or group and when those services were last accessed. User level report that shows the **"Last Accessed"** time for each service, indicating when that service was last used by the entity.
* IAM Access Analyser:&#x20;
* **Best practice:** use temporary security credentials (for IAM roles) instead of generated **access keys.**
* **Shared responsibility model for IAM:** User is responsible for MFA on accounts, appropriate permissions and policies.
*   **Best practice:** for root and IAM users to use both password + MFA, rotate access keys.

    * AWS provides guidelines for access key rotation every 90 days.



<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## IAM Policies ( Group, Inline)

* Policies can be attached to IAM users, groups, or roles. Give broad permissions.
* Groups <mark style="color:red;">can’t be nested</mark>, can contain users not groups.
* Granular principle of <mark style="color:red;">least privilege</mark>. Never grant more permissions than needed.
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

### Common use cases

* **AWS Lambda Execution Roles** ( IAM role to grant the function permissions to access other AWS services, like read/write to s3 bucket and access CloudWatch logs).
* **EC2 Role** (ex: IAM role with permissions to access s3  bucket without embedding access keys in the instance).
* AWS **Service Roles** ( Services assume the Role to perform actions on your behalf , for example, [**AWSCloud9ServiceRolePolicy**](#user-content-fn-1)[^1], which is AWS managed role with Service Linked Role Policy for AWS Cloud9. **AWSCloud9ServiceRolePolicy** is a managed policy attached to a service-linked role (often prefixed with `AWSServiceRoleFor`) which grants Cloud9 the necessary permissions to perform actions in your AWS account.

<details>

<summary><strong>AWSCloud9ServiceRolePolicy - AWS Managed</strong></summary>

```json
// Cloud9 standard aws maanged policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:RunInstances",
                "ec2:CreateSecurityGroup",
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceStatus",
                "cloudformation:CreateStack",
                "cloudformation:DescribeStacks",
                "cloudformation:DescribeStackEvents",
                "cloudformation:DescribeStackResources"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:TerminateInstances",
                "ec2:DeleteSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:DeleteStack"
            ],
            "Resource": "arn:aws:cloudformation:*:*:stack/aws-cloud9-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:instance/*",
                "arn:aws:ec2:*:*:security-group/*"
            ],
            "Condition": {
                "StringLike": {
                    "aws:RequestTag/Name": "aws-cloud9-*"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "ec2:ResourceTag/aws:cloudformation:stack-name": "aws-cloud9-*"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": [
                "arn:aws:license-manager:*:*:license-configuration:*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:ListInstanceProfiles",
                "iam:GetInstanceProfile"
            ],
            "Resource": [
                "arn:aws:iam::*:instance-profile/cloud9/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": [
                "arn:aws:iam::*:role/service-role/AWSCloud9SSMAccessRole"
            ],
            "Condition": {
                "StringLike": {
                    "iam:PassedToService": "ec2.amazonaws.com"
                }
            }
        }
    ]
}
```



</details>





* **Federated User** (For AD or SAML based Identity provider set up federation, allowing users to assume IAM roles in AWS to access resources based on their identity in the external system)
* **Cross-Account** Access
* **Break Glass** Emergency Roles ( on call engineers using emergency procedure to assume the role in case of A,B,C)&#x20;





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

[^1]: When you create an AWS Cloud9 environment, AWS automatically sets up the necessary service-linked role and attaches the appropriate policies for you
