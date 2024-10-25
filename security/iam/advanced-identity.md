# AWS STS

Service to grant limited-privileges credentials and temporary access to AWS resources (up to 1 hour).&#x20;

AWS STS supports AWS CloudTrail, a service that records AWS calls for your AWS account and delivers log files to an Amazon S3 bucket.

## API Actions

**AssumeRole** :black\_nib:**:** When you create a role, you create two policies: a role trust policy that specifies _who_ can assume the role, and a permissions policy that specifies _what_ can be done with the role.&#x20;

* This is the preferred approach when working with **temporary access** for specific permissions when you need to grant access to multiple users or external entities for a limited time.

```bash
aws sts get-caller-identity // get the account number ie 123456789012
aws sts assume-role 
--role-arn "arn:aws:iam::123456789012:role/MyTemporaryRole" 
--role-session-name "MySessionName"
```

To Assume Role.

1. Define an IAM Role
2. In the **trust policy** for the role, define the principal (user, role, or service) that can assume it. This principal can be within the same account or an external account.
3. Use STS API&#x20;

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::EXTERNAL_ACCOUNT_ID:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

```



**DecodeAuthorizationMessage** :black\_nib:**:** decode API error message.

**GetCallerIdentity** :black\_nib:**:** return this

```bash
    "UserId": "AIDASAMPLEUSERID",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/DevAdmin"
}
```

\
**GetSessionToken** :black\_nib:**:** when you want to **generate temporary credentials** for an existing IAM user without creating a new role or assuming a role. For example, short-lived credentials for an IAM user who already has specific permissions but wants to gain **temporary** elevated permissions or access AWS resources using **MFA**.

&#x20;`aws sts get-session-token --duration-seconds 86400 // for 24 hours`

To use with MFA:

1. Get token, calling `get-session-token`+ provide your **MFA device serial number** and **MFA token code.**
2. Configured IAM Policy with MFA condition

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1729843259425",
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::mybucketname",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "True"
        }
      }
    }
  ]
}
```



**AssumeRoleWithSAML:** returns short-term credentials for a role authenticated with SAML. This is **useful** when you have a company that uses an **external identity provider (IdP)**, like Microsoft ADy, Okta, or another SAML-based service, so instead of creating individual AWS IAM users for each person, you can set up **federated access.**&#x20;

This operation provides a mechanism for tying an enterprise identity store or directory to role-based AWS access without user-specific credentials or configuration

**GetFederationToken:** generate temporary credentials for federated user.\


### &#x20;Grant a user permissions to pass a role to an AWS service => a user must have permissions to _pass the role_ to the service with _iam:PassRole_&#x20;

To configure many AWS services, you must _pass_ an IAM role to the service. This allows the service to assume the role later and perform actions on your behalf. For most services, you only have to pass the role to the service once during setup, and not every time that the service assumes the role.

* To allow a user to pass a role to an AWS service, you must grant the `PassRole` permission to the user's IAM user, role, or group. This helps administrators ensure that only approved users can configure a service with a role that grants permissions.
* Create Trust Relationship  (policy) to allow traget service to assume it.&#x20;
* Some services automatically create a service-linked role in your account when you perform an action in that service.For example, Amazon EC2 Auto Scaling creates the `AWSServiceRoleForAutoScaling` service-linked role for you when you create an Auto Scaling group for the first time. If you try to specify the service-linked role when you create an Auto Scaling group and you don't have the `iam:PassRole` permission, you receive an error.&#x20;



For example: AWSServiceRoleForAWSCloud9 service role, AWS managed will have&#x20;

**Trus Relationship**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "cloud9.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

**Policy Attached -** AWSCloud9ServiceRolePolicy

```json
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



&#x20;\


#### Useful Links

[https://docs.aws.amazon.com/STS/latest/APIReference/API\_Operations.html](https://docs.aws.amazon.com/STS/latest/APIReference/API\_Operations.html)
