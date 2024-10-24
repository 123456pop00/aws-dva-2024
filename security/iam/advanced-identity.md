# AWS STS

Allows to grant limited and temporary access to AWS resources (up to 1 hour).&#x20;

• AssumeRole: Assume roles within your account or cross account\
• AssumeRoleWithSAML: return credentials for users logged with SAML\
• AssumeRoleWithWebIdentity

• return creds for users logged with an IdP (Facebook Login, Google Login, OIDC compatible...) • AWS recommends against using this, and using Cognito Identity Pools instead

• GetSessionToken: for MFA, from a user or AWS account root user\
• GetFederationToken: obtain temporary creds for a federated user\
• GetCallerIdentity: return details about the IAM user or role used in the API call\
• DecodeAuthorizationMessage: decode error message when an AWS API is denied

NOT FOR DISTRIBUTION © Stephane Maarek www.datacumulus.com

{% hint style="info" %}

{% endhint %}

##

```javascript
```

##
