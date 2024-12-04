---
icon: key
---

# AWS STS

Security Token Service is a web service that provides IAM and federated users with limited-privileges credentials and temporary access to AWS resources via API calls.&#x20;

STS issued credentials which are **not stored with user but are generated dynamically.**

* Allows to grant access to federated users without specifying IAM Identity for them on AWS.
* Cross-Account Access for multiple AWS accounts. Delegation approach.
* No need rotate, revoke long-term credentials because of the limited lifetime. Default Session Duration is 1 hour (3600 seconds).
* AWS STS supports CloudTrail, a service that records AWS calls for your account and delivers log files to an Amazon S3 bucket.

**Support for 2 Types of Federated Access via STS:**

1. **SAML 2.0** (Security Assertion Markup Language ) **IdPs**: Enterprise identity providers using SAML assertions. (SSO).
2.  **Web Identity** compatible with **OpenID Connect** (OIDC). Such as Salesforce, google, useful for mobile apps



## API Actions

**AssumeRole:** For an IAM user, same or cross account to assume an IAM Role, for example, dev role to assume privileged role to manage instances.  When you create role, it requires two policies: a role trust policy that specifies _who_ can assume the role, and a role permissions policy that specifies _what_ can be done with the role.&#x20;

This is the preferred approach when working with **temporary access** for specific permissions.

**AssumeRole API Call to STS:**

```bash
aws sts assume-role 
--role-arn "arn:aws:iam::123456789012:role/MyTemporaryRole" #required
--role-session-name "MySessionName" # required
--duration-seconds 86400 # optional 
```

**Returns:**

* **Access Key ID**
* **Secret Access Key**
* **Session Token**&#x20;

These must be loaded into env in order for assumed role to have the permissions attached in Permissions Policy.&#x20;

<details>

<summary><span data-gb-custom-inline data-tag="emoji" data-code="1f472">👲</span>  <mark style="color:red;">IAM user to assume role ( same/cross account)</mark></summary>

1. For user that will assume a role, add inline permissions / IAM policy. Permissions must be explicitly granted for this principal to assume a role.

```bash
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "AllowUserToAssumeSpecificRole",
			"Effect": "Allow",
			"Action": "sts:AssumeRole",
			"Resource": "arn:aws:iam::YOUR_ACCOUNT_ID:role/MyTemporaryRole"
		}
	]
}
```

2. Create Role to be assumed.
3. Create Role and Configure **Trust Policy** and **Permissions.**&#x20;
4. Trust Policy must have **arn** of the principal (user, role, or service) that can assume it. So Roles Trusted Entity will be the Account: 123456789012.&#x20;

```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/MyUserName"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

```

Multiple principles can be defined to assume a role.&#x20;

```bash
 "Principal": {
        "AWS": [
          "arn:aws:iam::22683702247:user/admin-user",
          "arn:aws:iam::22683702247:role/another-role",
          "arn:aws:iam::123456789012:user/external-user"
        ]
      },
```

5. Permissions Policy can be either inline or managed like AmazonEC2ReadOnlyAccess etc.
6. Configure CLI env

```bash
# configure
aws configure --profile sandbox
# switch
export AWS_PROFILE=sandbox
#check for env variables
env | grep AWS_ 
# AWS_PROFILE=sandbox
```

7. Run the API call to STS with **arn of the role** and set session name params.

This will  return JSON response in the terminal, which will look like this:

```json
{
  "Credentials": {
    "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
    "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYzEXAMPLEKEY",
    "SessionToken": "FQoGZXIvYXdzEFwa...",
    "Expiration": "2024-01-01T12:00:00Z"
  },
  "AssumedRoleUser": {
    "AssumedRoleId": "AROACLKWSDQRAEXAMPLE:YourSessionName",
    "Arn": "arn:aws:sts::123456789012:assumed-role/YourRoleName/YourSessionName"
  }
}

```

<mark style="color:red;">**Run assume-role and extract credentials.**</mark>

Below command extracts ready-to-use environment variable exports, allowing  to source the `credentials.properties` file to set temporary credentials in your environment with `source credentials.properties`

```bash
aws sts assume-role 
--role-arn "arn:aws:iam::ACCOUNT_ID:role/MyTemporaryRole" 
--role-session-name "MySessioName" 
--duration-seconds 7200
# extract the credentials from JSON reposnse & tee 
|sed 's/[," :]//g;s/AccessKeyId/export AWS_ACCESS_KEY_ID=/;s/SecretAccessKey/export AWS_SECRET_ACCESS_KEY=/;s/SessionToken/export AWS_SESSION_TOKEN=/' | grep 'export' | tee credentials.properties

```

If you define optional duration parameter, parameter ensure you update the session duration in management console (Select Role Summary -> Edit) or there will be (<mark style="color:red;">ValidationError)</mark> when calling the AssumeRole operation: The requested DurationSeconds exceeds the MaxSessionDuration set for this role.

![](<../../.gitbook/assets/Screenshot 2024-10-29 at 14.09.49.png>)

8. AssumeRole returns **Access Key ID**, **Secret Access Key**, and **Session Token** that are valid for the duration by default **1 hour or** set in duration parameter.&#x20;
9. Load those temporary credentials into env and verify

```bash
source credentials.properties
#verify
env | grep AWS_ 
# AWS_PROFILE=sandbox
# AWS_ACCESS_KEY_ID=DFGWHJFGKR
# AWS_SECRET_ACCESS_KEY=QAWSEDFGFEWEF
# AWS_SESSION_TOKEN=AQsdADFGdfghtrfgfr....
```

10. To remove credentials

```bash
unset AWS_ACCESS_KEY_ID AWS_SESSION....
```



</details>

<div align="left" data-full-width="true"><figure><img src="../../.gitbook/assets/image (4).png" alt="sts-iam-user-assumerole"><figcaption></figcaption></figure></div>

**AssumeRoleWithSAML:** returns short-term credentials for a role authenticated with SAML. This is **useful** when you have a company that uses an **external identity provider (IdP)**, like Microsoft AD, Okta, or another SAML-based service, so instead of creating individual AWS IAM users for each person, you can set up **federated access.**

This operation provides a mechanism for tying an enterprise identity store or directory to role-based AWS access without user-specific credentials or configuration.

**GetFederationToken:** generate temporary credentials for federated user. It returns a set of temporary security credentials ( access key ID, a secret access key, and a security token) for a user. A typical use is in a proxy application that gets temporary security credentials on behalf of distributed applications inside a corporate network. Must be called **with long-term security credentials of an IAM user**.

**DecodeAuthorizationMessage:** decodes API error message, giving additional information about the authorization status of a request from an encoded message returned in response to an AWS request. To decode an authorisation status message, a user must be granted permissions through an IAM permissions policy request the `DecodeAuthorizationMessage` (`sts:DecodeAuthorizationMessage`) action.

**GetCallerIdentity:** returns details about the IAM user or role whose credentials are used to call the operation.

**GetSessionToken:** to obtain temporary security credentials that provide enhanced security, particularly with Multi-Factor Authentication (MFA). This adds an additional layer of security by ensuring that only users who have access to the MFA device can obtain the temporary credentials.

1. Register MFA device for the user/role get device **arn**
2. **In CLA**&#x20;

`aws sts get-session-token --serial-number arn:aws:iam::123456789247:mfa/my-mfa-devicename --token-code 038969`

3. set up profile a`ws configure --profile mfa-enabled-profile,` if not set up add toke credentials into `nvim ~/.aws/credentials`
4. `export AWS_PROFILE=mfa-enabled-profile`

## Service Roles

<details>

<summary><span data-gb-custom-inline data-tag="emoji" data-code="1f510">🔐</span><mark style="color:red;">Grant a user permissions to pass a Role to an AWS service - iam:PassRole</mark> </summary>

To configure many AWS services, you must pass an IAM role to the service. This allows the service to assume the role later and perform actions on your behalf. For most services, you only have to pass the role to the service once during setup, and not every time that the service assumes the role.&#x20;

#### &#x20;User must have permissions to pass the role to the service with iam:PassRole

* To allow a user to pass a role to an AWS service, you must grant the `PassRole` permission to the user's IAM user, role, or group. This helps administrators ensure that only approved users can configure a service with a role that grants permissions.
* Create Trust Relationship (policy) to allow target service to assume it.
* Some services automatically create a service-linked role in your account when you perform an action in that service.For example, Amazon EC2 Auto Scaling creates the `AWSServiceRoleForAutoScaling` service-linked role for you when you create an Auto Scaling group for the first time. If you try to specify the service-linked role when you create an Auto Scaling group and you don't have the `iam:PassRole` permission, you receive an error.
* For example AWS managed AWSServiceRoleForAWSCloud9 service role comes with **Trust policy** that specifies the services allowed to assume the role:

&#x20;_**- cloud9.amazonaws.com** (Cloud9 itself) to act on your behalf to manage resources in your environment._

_**-ec2.amazonaws.com** because Cloud9 environments often use EC2 instances for compute resources._

And an AWS-managed policy **WSCloud9ServiceRolePolicy,** that grants the necessary permissions to set up and manage EC2 instances, security groups, and other resources required by Cloud9.

_As an **IAM user** or **IAM role** creating a Cloud9 environment, you need the `iam:PassRole` permission to allow Cloud9 to assume the `AWSServiceRoleForAWSCloud9` role on your behalf._

[_https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-reference-policy-checks.html#access-analyzer-reference-policy-checks-security-warning-pass-role-with-star-in-resource_](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-reference-policy-checks.html#access-analyzer-reference-policy-checks-security-warning-pass-role-with-star-in-resource)

![](../../.gitbook/assets/passRole-security.png)

</details>

### STS with MFA

1. **Attach MFA to IAM User**: Ensure the IAM user has MFA enabled. This can be done in the IAM Console.
2. To enforce MFA security layer, add a policy condition with boolean true for  `aws:MultiFactorAuthPresent` to restrict access based on whether MFA was used in the STS request.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:TerminateInstances",
        "ec2:DeleteSnapshot",
        "ec2:DeleteVolume"
      ],
      "Resource": "*",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}

```

_When attached to the IAM user or role it will restrict user or role that tries to delete EC2 instances to provide MFA, or the action will be denied._

2. **Get Session Token with MFA**: Use the `sts get-session-token` API call with the `--serial-number` and `--token-code` options.

* `--serial-number` is the ARN of the MFA device associated with the IAM user.
* `--token-code` is the 6-digit code from the MFA device.

```bash
aws sts get-session-token --serial-number arn:aws:iam::account-id:mfa/user-name --token-code 123456
```

3. **Receive Temporary Credentials**: AWS STS returns temporary security credentials **(Access Key ID, Secret Access Key, and Session Token**) that are valid for the specified duration (up to 36 hours with MFA).
4. **Use Temporary Credentials**: Save credentials in a file `credentials.properties` . Configure temporary credentials in your environment to make authenticated API requests.  Load into your shell environment by running: `source credentials.properties.`

```bash
# credentials file content
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_SESSION_TOKEN=your-session-token

```

#### Useful Links

[https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-options.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-options.html)

[https://docs.aws.amazon.com/IAM/latest/UserGuide/id\_roles\_providers\_saml\_3rd-party.html?icmpid=docs\_iam\_help\_panel\_create](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_saml_3rd-party.html?icmpid=docs_iam_help_panel_create)

[https://docs.aws.amazon.com/STS/latest/APIReference/API\_Operations.html](https://docs.aws.amazon.com/STS/latest/APIReference/API_Operations.html)
