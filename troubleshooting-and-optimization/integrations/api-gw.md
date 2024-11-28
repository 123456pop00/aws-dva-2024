# API GW

#### Key Features of API Gateway 📋

1. **Endpoint Management**: Allows you to create APIs that act as a "**proxy**" to your backend services.
   1. Support for stateful (WebSocket) and stateless (HTTP and REST) APIs.
2. **Security Features**: Supports IAM authentication, Lambda authorizers, and Amazon Cognito user pools. 🔒
   1. Integration with AWS WAF for protecting APIs against common web exploits&#x20;
3. **Throttling and Rate Limiting**: Protects your backend services from being overwhelmed by too many requests. ⚡
4. **Monitoring and Metrics**: Integrated with Amazon CloudWatch, so you can view API calls, latency, and error rates. 📊
   1. **Canary** release deployments for safely rolling out changes
   2. Allows to version your APIs, which can be useful for managing different versions of Lambda functions
   3. CloudTrail logging and monitoring of API usage and API changes&#x20;
5. **Caching**: Enhances performance and reduces the load on your backend using caching at the API Gateway layer. 🌀
6. **Flexible Integration Types**: Supports HTTP, HTTPS, and WebSocket protocols for real-time, two-way communication.
   1. Can define custom domain names and SSL/TLS certificates, providing a branded API endpoint

## Best Practices for Using API Gateway

1. Implement proper authentication and authorization mechanisms.
2. Use API keys or other forms of access control to secure your APIs.
3. Monitor API usage and performance using CloudWatch metrics.
4. Implement canary releases for safe deployment of new versions.
5. Use custom domain names for better branding and easier management.

## Lambda Integration&#x20;

* **Proxy Integration**: For simpler APIs where Lambda fully handles request/response logic.
* **Custom Integration**: For scenarios requiring transformations in the API Gateway itself (e.g., changing formats, adding headers, or supporting legacy APIs).

With **proxy integration**: RAW req. :on: RAW response&#x20;

<figure><img src="../../.gitbook/assets/lambda-get-proxy.png" alt=""><figcaption></figcaption></figure>

* API Gateway treats your Lambda function as a **black box** :black\_square\_button:
* It doesn't interfere with or modify the response returned by the Lambda function.
* All transformations (like changing the response format, headers, or status codes) must happen **within the Lambda function itself**.

<mark style="background-color:yellow;">If you want API Gateway to modify responses, you need to disable</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**proxy integration**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">and use</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**custom integration**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">instead.</mark>

* **Request Transformation**: Not possible. The raw API request is sent to Lambda.
* **Response Transformation**: Not possible. The raw Lambda response is returned to the client.

### Lambda&#x20;

What I get from API GW Event:

`export const handler = async (event) => { console.log("Received event from API GW", JSON.stringify(event, null, 2));`

```json
2024-11-28T12:22:20.049Z	7e184c15-1136-40f8-bf6a-279de9acc371	INFO	Received event from API GW 
{
    "resource": "/code-snippets",
    "path": "/code-snippets",
    "httpMethod": "GET",
    "headers": null,
    "multiValueHeaders": null,
    "queryStringParameters": null,
    "multiValueQueryStringParameters": null,
    "pathParameters": null,
    "stageVariables": null,
    "requestContext": {
        "resourceId": "9zvgia",
        "resourcePath": "/code-snippets",
        "httpMethod": "GET",
        "extendedRequestId": "B9QDYEwCgi0Fkug=",
        "requestTime": "28/Nov/2024:12:22:19 +0000",
        "path": "/code-snippets",
        "accountId": "060683702247",
        "protocol": "HTTP/1.1",
        "stage": "test-invoke-stage",
        "domainPrefix": "testPrefix",
        "requestTimeEpoch": 1732796539683,
        "requestId": "edcacaa0-d2ec-4c9e-91c8-aa64363a4e6a",
        "identity": {
            "cognitoIdentityPoolId": null,
            "cognitoIdentityId": null,
            "apiKey": "test-invoke-api-key",
            "principalOrgId": null,
            "cognitoAuthenticationType": null,
            "userArn": "arn:aws:iam::060683702247:user/juliatorp",
            "apiKeyId": "test-invoke-api-key-id",
            "userAgent": "",
            "accountId": "123456789",
            "caller": "AIDAQ4IIHZ7TSTRFZ7YM4",
            "sourceIp": "test-invoke-source-ip",
            "accessKey": "ASIAQ4IIHZ7TS3K2VKRH",
            "cognitoAuthenticationProvider": null,
            "user": "AIDAQ4IIHZ7TSTRFZ7YM4"
        },
        "domainName": "testPrefix.testDomainName",
        "apiId": "amzefzxc47"
    },
    "body": null,
    "isBase64Encoded": false
}
```

API GW Stage Versioning +  Lambda Aliases -> **maintain backward compatability for old API version when we introduced breaking changes** :scream\_cat:

1. Getting prod / dev aliases of lambda when we create method / integration request for lambda, its ARN will mention alias for mapping to specific stage .

`arn:aws:lambda:eu-north-1:05664565247:function:lambda-function-name:${stageVariables.lambdaAlias}`

* arn:aws:lambda:eu-north-1:060683702247:function:lambda-foo:DEV
* arn:aws:lambda:eu-north-1:060683702247:function:lambda-foo:UAT

_You defined your Lambda function as a stage variable. Run the following AWS CLI command to ensure you have the appropriate policy for this function. Replace the stage variable in the function-name parameter with the necessary function name.Add permission command_

**Allow DEV alias to invoke function**

<figure><img src="../../.gitbook/assets/stage-vars.png" alt=""><figcaption></figcaption></figure>

```bash
aws lambda add-permission \
#-function-name "arn:aws:lambda:eu-north-1:0102030405067:function:lambda-foo:UAT" \
--function-name "arn:aws:lambda:eu-north-1:0102030405067:function:lambda-foo:DEV" \
--source-arn "arn:aws:execute-api:eu-north-1:0102030405067:amzefzxc47/*/GET/stage-variables" \
--principal apigateway.amazonaws.com \
--statement-id b7a1d265-018d-4fdb-9c94-73dda8f879a0 \
--action lambda:InvokeFunction
```

Will allow to test

<figure><img src="../../.gitbook/assets/test-api-gw-lambdaAlias.png" alt=""><figcaption></figcaption></figure>

### Canary Deployment



{% tabs %}
{% tab title="Canary🦜" %}
**Definition**: A deployment strategy that allows routing a <mark style="background-color:yellow;">small percentage of traffic</mark> to a new version of your API Gateway stage configuration while keeping the majority of traffic on the current version.

* **Key Feature**: Canary deployments can test incremental changes without replacing the entire stage.
* **Focus**: Canary deployments test API Gateway-specific features like:
  * Request/response mappings.
  * New stage variables or settings.
  * Integration with multiple services (e.g., Lambda, Step Functions).
  * Caching or throttling configurations.
* **Traffic Handling**: Splits API traffic by percentage, e.g., 10% to the canary version and 90% to the current stable version.
{% endtab %}

{% tab title="Blue 🦋 /Green 🐸 " %}
**Definition**: A deployment strategy where two environments exist <mark style="background-color:blue;">simultaneously</mark>—"blue" (current/stable) and "green" (new/testing). Traffic is gradually shifted from blue to green, allowing a complete rollback if issues arise.

* **Key Feature**: Blue-green deployments with Lambda often leverage **Lambda aliases** and **weighted routing** for traffic splitting.
* **Focus**: Targets **Lambda function code and configuration updates**, such as:
  * Function logic changes.
  * Memory allocation or timeout adjustments.
  * Integration at the function level, not API-wide.
* **Traffic Handling**: Splits traffic at the function level, directing portions to different Lambda versions via weighted aliases.
{% endtab %}
{% endtabs %}

#### **When to Use Canary vs. Blue-Green?**

| **API-wide changes**               | ✅ Yes | ❌ No  |
| ---------------------------------- | :---: | ----- |
| **Single Lambda update**           |  ❌ No | ✅ Yes |
| **End-to-end integration testing** | ✅ Yes | ❌ No  |
| **Function-specific testing**      |  ❌ No | ✅ Yes |

**When to use:**

* When the change impacts the entire API Gateway configuration or stage settings.
* When testing involves multiple integrated resources (e.g., Lambda, Step Functions, DynamoDB).
* For validating broader, end-to-end API changes that span the infrastructure.

#### **API Gateway Canary Deployments:**

* **Scope**: Operates at the API Gateway stage level. Divides traffic between API deployments.&#x20;
* **Use Case**: Allows you to roll out a new version of the **entire API** to a subset of users. Backward compatabiltiy for breaking changes.
* **Control**:
  * Canary settings can route a percentage of API traffic (e.g., 90/10) to a new deployment in the **canary stage**.
  * This includes all resources integrated with the API, not just a single Lambda function.
* **Benefits**:
  1. **End-to-End Testing**: Canary deployments can test changes in the broader system (e.g., changes in API Gateway stage configuration, Lambda function, IAM policies, etc.).
  2. **Validation of API-level changes**: It tests everything that happens **before** the Lambda is invoked, such as:
     * Request/response transformations.
     * API Gateway throttling or caching changes.
  3. **Incorporates Multiple Backends**: If your API Gateway stage integrates with multiple backends (e.g., Lambda, DynamoDB, Step Functions), the canary rollout spans the entire infrastructure.
* **Limitation**: The focus is broader than function-level updates, and it requires managing deployments at the API Gateway stage level.



#### Deploying to Canary:

1. First configure arn: version1  on resource like /featured-blogs&#x20;
2. Deploy APIs to 'canary\_stage'
3. Reconfigure /featured-blogs methods to point to lambda arn: version2 + Deploy
4. configure Canary

<div><figure><img src="../../.gitbook/assets/canary-test.png" alt=""><figcaption></figcaption></figure> <figure><img src="../../.gitbook/assets/promote-canary.png" alt=""><figcaption></figcaption></figure></div>



