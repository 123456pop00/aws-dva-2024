---
description: All cmd are for mac os
---

# Useful CLI

In console create user with programatic access only.

Folder: `~/.aws contains` \~/.aws/credentials and \~/.aws/config.

#### Credentials File&#x20;

It is used by the AWS CLI and SDKs to authenticate requests. Not very responsibly `aws configure` writes IAM credentials to the `~/.aws/credentials` file in <mark style="color:red;">plain text.</mark>&#x20;

```bash
[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY

[profile1]
aws_access_key_id = YOUR_ACCESS_KEY_ID_1
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY_1

[profile2]
aws_access_key_id = YOUR_ACCESS_KEY_ID_2
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY_2

```

#### Config file

Contains configuration settings for profiles.

```bash
[default]
region = us-west-2
output = json

[profile profile1]
region = us-east-1
output = text

[profile profile2]
region = eu-central-1
output = json

```

#### **Common environment variables include:**

* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `AWS_SESSION_TOKEN` (for temporary credentials)
* `AWS_REGION`
* `AWS_PROFILE` (to specify which profile to use)

Set up  profiles:

```bash
$ aws configure --profile sandbox
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: eu-west-1
Default output format [None]: json
```

List profiles:

```bash
 aws configure list-profiles
```

Switching profile :

```
aws s3 ls --profile sandbox
aws aws ec2 describe-instances --profile sandbox
```

or&#x20;

```bash
#check
evn | grep AWS
# set
export AWS_PROFILE=sandbox 
```

Shows which profile is used for AWS CLI operations&#x20;

```bash
aws configure list
```

For example:

<div align="left">

<figure><img src="../.gitbook/assets/Screenshot 2024-10-28 at 15.34.27.png" alt="" width="375"><figcaption></figcaption></figure>

</div>

Get the profile details:

```sh
aws sts get-caller-identity
```

{

&#x20;   `"UserId": "AKIAIOSFODNN7EXAMPLE`"`,`

&#x20;   `"Account": "123576896797",`

&#x20;   `"Arn": "arn:aws:iam::123576896797:user/sandbox-user"`

`}`

Remove env variables :

```bash
unset AWS_ACCESS_KEY_ID
```

Generate temp credential for local testing, 3rd party access, Multi-Factor Authentication (MFA) Enforcement

```bash
aws sts get-session-token --duration-seconds 3600

```





