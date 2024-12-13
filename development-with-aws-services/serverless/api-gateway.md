---
icon: swap-arrows
---

# API Gateway

> API Gateway is a service that facilitates the creation, publishing, maintenance, monitoring, and security of your APIs at any scale. &#x20;
>
> * You only pay when your APIs are in use
>   * flat charge per million API Gateway requests
>   * hourly rate for each stage's provisioned cache&#x20;
> * &#x20;traffic management
> * api Versioning
> * Stages are similar to tags. They define the path through which the deployment is accessible.
> * api keys + usage plans
> * handles authentication + authorisation&#x20;
> * rate limiting +  throttling, monitoring
> * generates SDK -> API Gateway can generate client SDKs for Java, JavaScript, Java for Android, Objective-C or Swift for iOS, and Ruby. You can use AWS Command Line Interface (AWS CLI) to generate and download an SDK of an API for a supported platform by calling the **get-sdk** command
> * WAF integration
> * **Cache** frequently accessed responses (only GET )  so that subsequent requests don't go to backend, can be configured per stage,  has 5min TTL, can be **invalidated with proper IAM + Cache-Control header max-size=0**
>   * **charged at an hourly rate**
>   * can use parameters in the method to form cache keys so that API Gateway caches the method's responses depending on the parameter values used
>   * Two ways to verify caching:
>     1. CloudWatch Metrics: **CacheHitCount** and **CacheMissCount**.
>     2. Create a timestamp and include it in your API response.
> * import a REST API from an external definition file -  OpenAPI (Swagger™ is a project used to describe and document RESTful APIs.)
> * **transform and validate request response**&#x20;

## User Authorisation + management

* You can authorize access to your APIs with **IAM** :man\_office\_worker: and Amazon **Cognito or custom**
* :key: API keys for developers -> You can create API keys on API Gateway, set fine-grained access permissions on each API key ( its not a primary authorization mechnisam it **tracks usage**)

## Backend integrations -> endpoitns

#### The API integration has an integration request and an integration response option.

#### API endpoint

A hostname for an API in API Gateway that is deployed to a specific Region. The hostname is of the form `{api-id}.execute-api.{region}.amazonaws.com`. **The following types of API endpoints are supported:**

#### **Edge optimised** ( ACM from us-east-1, api service still in one regions)

1. designed to help you <mark style="background-color:yellow;">reduce client latency from anywhere on the internet</mark>
2. API Gateway **will automatically configure a fully managed CloudFront** distribution to provide lower latency, :star: you don’t have to pay for or manage a CDN separately

<figure><img src="../../.gitbook/assets/edge-endpoint.png" alt=""><figcaption></figcaption></figure>

#### **Regional** (requests are targeted directly to the Region-specific API Gateway API without going through any CloudFront distribution)

1. designed to reduce latency when calls are made from the same AWS Region as the API, for example, an API that is going to be accessed from EC2 instances within the same Region
2. does not deploy its own CloudFront distribution in front of your API, you have flexibility to **deploy your own CloudFront** distribution or content delivery network (CDN) in front of API Gateway and control that distribution using your own settings for customized scenarios

<figure><img src="../../.gitbook/assets/regional-endpoint.png" alt=""><figcaption></figcaption></figure>

#### **Private** (access only via VPC ENI)

1. designed to expose APIs only inside your selected VPC
2. designed for applications that have very secure workloads, such as healthcare or financial data that cannot be exposed publicly on the internet
3. PrivateLink charges apply&#x20;
4. <mark style="background-color:red;">change from private to edge-optimized is not a supported endpoint change.</mark>

<figure><img src="../../.gitbook/assets/private-endpoint.png" alt=""><figcaption></figcaption></figure>

### **5 API Gateway integration types**

-> determines **how method request data is passed to the backend.** As part of creating the method, you must choose an integration type&#x20;

1. Lambda
2. HTTP&#x20;
   1. if proxy is not configured, you’ll need to configure both the integration request and the integration response&#x20;
   2. &#x20;just proxy
3. AWS service, like drop a message directly into an Amazon Simple Queue Service (Amazon SQS) queue.
4. Mock - return a response without sending the request further to the backend, This is a good idea for a health check endpoint to test your API. Anytime you want a hardcoded response to your API call, use a Mock integration.
5. VPC  Link - endpoint on your EC2 instance that’s not public. API Gateway can’t access it unless you use the VPC link and you have to have a Network Load Balancer on your backend.



<div align="left"><figure><img src="../../.gitbook/assets/apiGW-integrations.png" alt="" width="356"><figcaption></figcaption></figure></div>

## 2 options to create RESTful APIs—REST APIs and HTTP APIs

### REST&#x20;

* REST APIs offer API proxy functionality and API management features in a single solution. REST APIs also offer API management features such as usage plans, API keys, publishing, and monetizing APIs
* all-inclusive stateless with certififcates

### HTTP

* HTTP APIs are optimized for building APIs that proxy to Lambda functions or HTTP backends, making them ideal for serverless workloads.&#x20;
* No management functionality.
* Only **regional endpoint type**
* equipped with native OIDC and OAuth 2 authorization

## WebSocket APIs :electric\_plug: :stars:

* With WebSocket APIs in API Gateway, you can define backend integrations with Lambda functions, Amazon Kinesis, or any HTTP endpoint to be invoked when messages are received from the connected clients.
* Non-JSON messages are directed to a **$default** route that you configure. Route includes a route key, which is the value that is expected once a route selection expression is evaluated.&#x20;
* The route selection expression is an attribute defined at the API level. It specifies a JSON property that is expected to be present in the message payload. Like $greet $play -> you must integrate it with an endpoint in the backend -> JSON messages can be routed to invoke a specific backend service based on message content
* **Custom route** like **$join** ->  When an incoming message contains a JSON property, and that property evaluates to a value that matches the route key value, API Gateway invokes the integration.
* There are three predefined routes that can be used with WebSocket APIs: **$connect**, **$disconnect**, and **$default**.
  * $default route keys, to specify a fallback route. For example, this could be for a generic mock integration that returns a particular error message for incoming messages that don't match any of the defined route keys.
  * You can use $default without any defined route keys, to s**pecify a proxy model that delegates routing to a backend component.**
  * Use it to specify a route for non-JSON payloads.

<figure><img src="../../.gitbook/assets/apiGW-wss.png" alt=""><figcaption></figcaption></figure>

## Deploying&#x20;

<mark style="background-color:red;">When you are ready to make your API callable for your users, you need to deploy your API to a stage</mark>

* When you deploy your API, you deploy to a **stage its like a snapshot of current api**&#x20;
* Use stages with **canary deployments to test new versions.**
* Anytime you update anything about the API, you need to redeploy it to an existing stage or to a new stage that you create as part of the deploy action
* **Support for custom hostnames,** API Gateway is integrated with ACM and lets you import your own certificate or generate a SSL certificate with ACM
* That base URI is called the _invoke URL_, and its composition will look like this:

<figure><img src="../../.gitbook/assets/apiGW.png" alt=""><figcaption></figcaption></figure>

#### HTTP proxy or  Lambda proxy option

* A proxy resource is expressed by a special resource path parameter of **{proxy+}**, often referred to as a greedy path parameter. The plus sign (+) indicates child resources appended to it.

### Testing&#x20;

**Response logs**

Test results include simulated CloudWatch logs. No data is actually written to CloudWatch when testing. Although logs are not generated, **but the APIs are actually executed.**



**Latency:** Latency is the time between the receipt of the request from the caller and the returned response.

### **Simplify version management with stage variables -> r**etrieved dynamically at runtime

As you define variables in the stage settings in the console, you can reference them with the **$stageVariables.\[variable name]** notation.&#x20;

You can also **inject stage-dependent items at runtime such as:**

* URLs
* Lambda functions -> **use stages with Lambda aliases**&#x20;
* Any necessary variables

### **Canary deployments**

* &#x20;send a percentage of traffic to your "canary" while leaving the bulk of your traffic on a known good version of your API until the new version has been verified.

<figure><img src="../../.gitbook/assets/apiGW-canary.png" alt=""><figcaption></figcaption></figure>

### SAM&#x20;

-> best practices for deploying your APIs and serverless applications to production using AWS SAM as your application framework.











