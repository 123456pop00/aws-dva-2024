---
icon: user-secret
---

# System Manager

## Application Tools -> PArameter Store

## Lambda +  SSM Client instance

* Lambda must have  appropriate IAM permissions to access SSM parameter -> **inline policy to have this for SSM + KMS ( decrypt)**

```jsonp
// Allow lambda to use demo api key in dev env
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ssm:GetParametersByPath",
                "ssm:GetParameters",
                "ssm:GetParameter",
                "kms:Decrypt"
            ],
            "Resource": [
                "arn:aws:ssm:eu-north-1:<ACCOUNT_ID>:parameter/my-bot-app/dev/*",
                "arn:aws:kms:eu-north-1:<ACCOUNT_ID>:key/0c251475-45c6-456b-b093-8bc80a3e83441"
            ]
        }
    ]
}
```



* **WithDecryption**: Ensure `WithDecryption: true` if you’re fetching a <mark style="background-color:red;">SecureString</mark> parameter.

```
1. SSM  ERROR Error retrieving parameter: AccessDeniedException: User: arn:aws:sts::<ACCOUNT_ID>:assumed-role/ssm-parameters-lambda-role-xv550iyn/ssm-parameters-lambda is not authorized to perform: ssm:GetParameter on resource: arn:aws:ssm:eu-north-1:<ACCOUNT_ID>:parameter/bot-api because no identity-based policy allows the ssm:GetParameter action
2. KMS ERROR Error retrieving parameter: AccessDeniedException: User: arn:aws:sts::<ACCOUNT_ID>:assumed-role/ssm-parameters-lambda-role-xv550iyn/ssm-parameters-lambda is not authorized to perform: kms:Decrypt on resource: arn:aws:kms:eu-north-1:<ACCOUNT_ID>:key/0c251475-45c6-456b-b093-8bc80a3e83441 because no identity-based policy allows the kms:Decrypt action (Service: AWSKMS; Status Code: 400; Error Code: AccessDeniedException; Request ID: e60ff77a-9a25-4de1-b5c7-745dd131923e; Proxy: null)

```



