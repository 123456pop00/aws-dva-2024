# AWS STS

AWS Security Token Service is a web service that provides IAM and federated users with limited-privileges credentials and temporary access to AWS resources via API calls.

STS issued credentials which are not stored with user but are generated dynamically.&#x20;

* Allows to grant access to federated users without specifying IAM Identity for them on AWS.&#x20;
* Cross-Account Access for multiple AWS accounts. Delegation approach.
* No need rotate, revoke long-term credentials because of the limited lifetime. Default Session Duration is 1 hour (3600 seconds).

#### Support for 2 Types of Federated Access via STS:

* **SAML 2.0** (Security Assertion Markup Language ) **IdPs**: Enterprise identity providers using SAML assertions. (SSO)&#x20;
* **Web Identity** compatible with **OpenID Connect** (OIDC). Such as Salesforce, google, useful for mobile apps.

<div align="left">

<figure><img src="../../.gitbook/assets/Screenshot 2024-10-28 at 13.59.26.png" alt="" width="188"><figcaption></figcaption></figure>

</div>

* AWS STS supports CloudTrail, a service that records AWS calls for your  account and delivers log files to an Amazon S3 bucket.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## API Actions

**AssumeRole:** When you create a role, you create two policies: a role trust policy that specifies _who_ can assume the role, and a role permissions policy that specifies _what_ can be done with the role.&#x20;

* This is the preferred approach when working with **temporary access** for specific permissions when you need to grant access to multiple users or external entities.

For IAM user to make API Call to STS, a user must provide:&#x20;

```bash
aws sts get-caller-identity // get the account number ie 123456789012
aws sts assume-role 
--role-arn "arn:aws:iam::123456789012:role/MyTemporaryRole" 
--role-session-name "MySessionName"
```

STS  AssumeRole returns:

**Access Key ID**, **Secret Access Key**, and **Session Token** that are valid for the specified duration.

To Assume Role:

1. Define an IAM Role.
2. In the **trust policy** for the role, define the principal (user, role, or service) that can assume it. This principal can be within the same account or an external account.

{% code title="Trust Policy " %}
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
{% endcode %}



**DecodeAuthorizationMessage:** decode API error message.

**GetCallerIdentity:** return this :arrow\_down:

```bash
    "UserId": "AIDASAMPLEUSERID",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/DevAdmin"
}
```

\
**GetSessionToken:** when you want to **generate temporary credentials** for an existing IAM user without creating a new role or assuming a role. For example, short-lived credentials for an IAM user who already has specific permissions but wants to gain **temporary** elevated permissions or access AWS resources using **MFA**.

&#x20;`aws sts get-session-token --duration-seconds 86400 // for 24 hours`

To use with MFA:

1. Get token, calling `get-session-token`+ provide your **MFA device serial number** and **MFA token code.**
2. Configured IAM Policy with MFA condition

```json
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



**AssumeRoleWithSAML:** returns short-term credentials for a role authenticated with SAML. This is **useful** when you have a company that uses an **external identity provider (IdP)**, like Microsoft AD, Okta, or another SAML(Security Assertion Markup Language)-based service, so instead of creating individual AWS IAM users for each person, you can set up **federated access.**&#x20;

This operation provides a mechanism for tying an enterprise identity store or directory to role-based AWS access without user-specific credentials or configuration.

**GetFederationToken:** generate temporary credentials for federated user.\


### :closed\_lock\_with\_key:Grant a user permissions to pass a role to an AWS service => a user must have permissions to _pass the role_ to the service with _iam:PassRole_&#x20;

To configure many AWS services, you must _pass_ an IAM role to the service. This allows the service to assume the role later and perform actions on your behalf. For most services, you only have to pass the role to the service once during setup, and not every time that the service assumes the role.

* To allow a user to pass a role to an AWS service, you must grant the `PassRole` permission to the user's IAM user, role, or group. This helps administrators ensure that only approved users can configure a service with a role that grants permissions.
* Create Trust Relationship  (policy) to allow traget service to assume it.&#x20;
* Some services automatically create a service-linked role in your account when you perform an action in that service.For example, Amazon EC2 Auto Scaling creates the `AWSServiceRoleForAutoScaling` service-linked role for you when you create an Auto Scaling group for the first time. If you try to specify the service-linked role when you create an Auto Scaling group and you don't have the `iam:PassRole` permission, you receive an error.&#x20;



For example: AWSServiceRoleForAWSCloud9 service role, AWS managed will have&#x20;

**Trust Relationship**

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

_**AWSCloud9ServiceRolePolicy**_** Policy Attached**&#x20;

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

### IAM user to assume role

1. User must have permissions policy for the Role

```bash
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "AssumeSandboxUser",
			"Effect": "Allow",
			"Action": "sts:AssumeRole",
			"Resource": "arn:aws:iam::YOUR_ACCOUNT_ID:role/NameOfTheRole_ToBe_Assumed"
		}
	]
}
```

2. Configure Role in the same or another account that user will assume.
3. **Create Trust policy** that has the ARN of the User that should be able to assume this role.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::YOUR_ACCOUNT_ID:user/USER_NAME"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

```

3. **Attach Permissions Policy** to the role. Can be managed on inline.
4. **CLI Configuration**: Configure the  user in your CLI using:&#x20;

```bash
# configure
aws configure --profile profileName
# switch
export AWS_PROFILE=profileName
```

5. **Call AssumeRole**: Use the `--role-arn` and `--role-session-name` parameters, with optional `--duration-secondss`

If you change `--duration-seconds 7200` parameter ensure you update the session duration in management console (Select Role Summary -> Edit)

An error occurred (<mark style="color:red;">ValidationError</mark>) when calling the AssumeRole operation: The requested DurationSeconds exceeds the MaxSessionDuration set for this role.

<div align="left">

<figure><img src="../../.gitbook/assets/Screenshot 2024-10-28 at 17.36.01.png" alt="" width="375"><figcaption></figcaption></figure>

</div>

6. Run AssumeRole and pipe credentials into a file

```bash
aws sts assume-role 
--role-arn "arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME" 
--role-session-name "SESSION-NAME" | sed 's/[," :]//g;s/AccessKeyId/export AWS_ACCESS_KEY_ID=/;s/SecretAccessKey/export=AWS_SECRET_ACCESS_KEY=/;s/SessionToken/export AWS_SESSION_TOKEN=/ ' | grep 'export' | tee credentials.properties
```



7. Load extracted credentials into environment with . `credentials.properties` or `source credentials.properties`



#### Useful Links

[https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-options.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-options.html)

[https://docs.aws.amazon.com/IAM/latest/UserGuide/id\_roles\_providers\_saml\_3rd-party.html?icmpid=docs\_iam\_help\_panel\_create](https://docs.aws.amazon.com/IAM/latest/UserGuide/id\_roles\_providers\_saml\_3rd-party.html?icmpid=docs\_iam\_help\_panel\_create)

[https://docs.aws.amazon.com/STS/latest/APIReference/API\_Operations.html](https://docs.aws.amazon.com/STS/latest/APIReference/API\_Operations.html)
