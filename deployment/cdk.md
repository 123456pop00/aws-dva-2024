# CDK

* Use  JavaScript/TypeScript, Python, Java and .NET as  **IaC to define infrastructure** vs. declarative config files as templates in CF or it's serverless SAM.&#x20;
  * Leverages CF as code is compiled into CF template (JSON/YAML) using **CDK CLI**
  * Great for Lambda functions and Docker containers
  * Contains high-level components called **constructs**
  * Combine SAM + CDK by using synth to test CDK apps, i.e., we have lambda function in CDK but we run **synth command what will generate CF template -> then we use SAM to test locally**&#x20;
    * `my-application$ :sam build`
    * `my-application$: sam loc`
    * `al invoke putItemFunction --event events/event-post-item.json`

## Constructs

> Component that encapsulates everything CDK needs to create the final CloudFormation stack. It is a **cloud resource or a logical unit** that encapsulates resources, such as a Lambda, an S3 bucket, or a DynamoDB table, and provides an abstraction layer to make it easier to define and manage infrastructure as code.
>
> Constructs are designed to be **composed** together. You can combine smaller constructs into more complex ones. For example, you can combine a Lambda function, an API Gateway, and a DynamoDB table to create a serverless backend.

### **Hierarchy of Constructs - 3 LEVELS: patterns, boilerplate, raw cFn**:

Constructs in CDK are organised in a **hierarchical tree structure**. A **Construct Tree** is composed of "constructs" that can be composed of other constructs.

:european\_castle: **Level 3 (L3) &#x20;**_**aka**_**&#x20;High-Level Patterns and Architectural Constructs:** These are fully abstracted "patterns" for common use cases, such as creating a complete web application or a serverless stack.

* Collection of multiple related AWS resources forming a pattern (high level)
* CDK provides Level 3 constructs **in libraries** like `aws-cdk-lib/aws-apigateway` (for API Gateway with Lambda backends), `aws-cdk-lib/aws-dynamodb` (for pre-configured DynamoDB tables), and `aws-cdk-lib/aws-s3` (for advanced integrations with Lambda triggers).
* Level 3 stacks implement common architectural patterns, such as **serverless applications**, **REST APIs**, **event-driven architectures**
  * **Handles orchestration**: Often automatically handles integration between different AWS services like Lambda, API Gateway, DynamoDB, S3, SNS, and more.
* Writing difficult CF templates like `aws-ecs-patterns.ApplicationLoadBalancerFargateService` as an architecture that includes a Fargate cluster with ALB will be easier with cdk

```javascript
// rest pattern
const cdk = require('aws-cdk-lib');
const { Stack, Construct } = cdk;
const { LambdaRestApi } = require('aws-cdk-lib/aws-apigateway');
const { Function } = require('aws-cdk-lib/aws-lambda');
const { Table } = require('aws-cdk-lib/aws-dynamodb');

class ServerlessApiStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Create DynamoDB Table (Level 2)
    const table = new Table(this, 'ItemsTable', {
      partitionKey: { name: 'id', type: ddb.AttributeType.STRING },
      sortKey: { name: 'timestamp', type: ddb.AttributeType.STRING },
      billingMode: ddb.BillingMode.PAY_PER_REQUEST,
    });

    // Create Lambda function (Level 2)
    const lambdaFunction = new Function(this, 'MyLambda', {
      runtime: 'nodejs16.x',
      handler: 'index.handler',
      code: Function.fromInline(`
        exports.handler = async (event) => {
          // Lambda code to interact with DynamoDB and return a response
          return {
            statusCode: 200,
            body: JSON.stringify({ message: 'Hello World' }),
          };
        };
      `),
      environment: {
        TABLE_NAME: table.tableName,
      },
    });

    // Grant Lambda function permissions to access DynamoDB table
    table.grantReadWriteData(lambdaFunction);

    // Create REST API (Level 3 abstraction - combines API Gateway and Lambda)
    new LambdaRestApi(this, 'MyRestApi', {
      handler: lambdaFunction,
      proxy: false, // We specify methods
      deployOptions: {
        stageName: 'prod',
      },
    });
  }
}

module.exports = { ServerlessApiStack };

```



:house: **Level 2 (L2)&#x20;**_**aka**_**&#x20;Boilerplate or Service-Specific Constructs**: These are higher-level abstractions that wrap around L1 constructs. They come with sensible defaults and are generally easier to work with. They're  built on top of the Level 1 constructs, that aim to simplify the resource setup and usage bu providign boilerplate to resource defualt properties.

```javascript
  // Create DynamoDB table
    const xMasMoviesTable = new ddb.Table(this, 'XMasMovies', {
      partitionKey: { name: 'id', type: ddb.AttributeType.STRING },
      sortKey: { name: 'timestamp', type: ddb.AttributeType.STRING },
      billingMode: ddb.BillingMode.PROVISIONED,
      readCapacity: 2,
      writeCapacity: 2,
    });

    // Create S3 bucket with versioning enabled
    const xMasMoviesBucket = new s3.Bucket(this, 'XMasMoviesBucket', {
      versioned: true,
      removalPolicy: cdk.RemovalPolicy.DESTROY, // To remove bucket on stack deletion
    });

```



#### :bricks: **Level 1 (L1) &#x20;**_**aka**_**&#x20;Cfn classes or Raw CF Resources:**&#x20;

These are low-level constructs that map directly to CloudFormation resources, so they are the **raw** representations of CloudFormation resources. They provide the **most flexibility** but require you to specify most of the resource properties.

* Construct names start with `Cfn` prefix, which stands for CloudFormatio&#x6E;**.** _For example, `CfnBucket` represents an S3 bucket, and `CfnFunction` represents a Lambda function._
*   Must explicitly configure all resource properties

    * This allows for most fine-grained control over your AWS resources since you define all their properties manually.


* In **Level 1**, you directly use the AWS CloudFormation resources (e.g., `AWS::Lambda::Function`, `AWS::S3::Bucket`

**Examples of L1 constructs**

```javascript
// table + s3bucket
import * as cdk from 'aws-cdk-lib';
import * as ddb from 'aws-cdk-lib/aws-dynamodb';

const xMasMoviesTableCfn = new ddb.CfnTable(this, 'XMasMovies', {
  tableName: 'x-mas-movies', // Optional, you can provide a custom table name
  keySchema: [
    {
      attributeName: 'id',
      keyType: 'HASH', // Partition key
    },
    {
      attributeName: 'timestamp',
      keyType: 'RANGE', // Sort key
    },
  ],
  attributeDefinitions: [
    {
      attributeName: 'id',
      attributeType: 'S', // 'S' for String type
    },
    {
      attributeName: 'timestamp',
      attributeType: 'S', // 'S' for String type
    },
  ],
  provisionedThroughput: {
    readCapacityUnits: 2,
    writeCapacityUnits: 2,
  },
});

 
// s3 bucket
import * as s3 from 'aws-cdk-lib/aws-s3';

const xMasMoviesBucketCfn = new s3.CfnBucket(this, 'XMasMoviesBucket', {
  versioningConfiguration: {
    status: 'Enabled', // Versioning enabled
  },
  deletionPolicy: 'Delete', // Equivalent to removalPolicy: cdk.RemovalPolicy.DESTROY
});

```





## CDK Commands

#### 1. **Install CDK** 📦

```bash
bashCopy codesudo npm install -g aws-cdk-lib
```

#### 2. **Set Up Your Project** 📁

```bash
mkdir cdk-app
cd cdk-app/
```

#### 3. **Initialize CDK Application** 🚀

```bash
cdk init app --language javascript
```

```bash
 cdk ls
```

Get detailed info on resources defined in your CDK stack.&#x20;

```bash
cdk describe
```

#### 4. **Copy Stack File** 📝

Copy your stack logic into the `lib/cdk-app-stack.js` file (this will contain your resources).

#### 5. **Set Up Your Lambda Function** 🐍

Create a Lambda directory and add a function file (replace `index.py` with your Lambda logic file):

```bash
mkdir lambda && cd lambda
touch index.py
```

#### 6. **Bootstrap Your Environment** 🔧

Bootstrapping is needed to set up the environment for deploying CDK stacks (e.g., S3 buckets for deployment artifacts) and If you need to re-bootstrap or update resources (like when switching AWS accounts or regions)

```bash
cdk bootstrap
```

#### 7. **Synthesize CloudFormation Template** 💻

This is optional but allows you to see the CloudFormation template generated by the CDK before you deploy:

```bash
cdk synth
#Before Deployment,check for errors or potential issues 🚫
cdk validate
```

#### 8. **Deploy the Stack** 🌐

Deploy the CDK stack to AWS:

```bash
cdk deploy
#Compare your local stack with the deployed stack.
cdk diff

```

#### 9. **Empty Your S3 Bucket** 🧹

(Optional) If you want to clean up your S3 bucket or resources:

```bash
# Use the AWS CLI or CDK to empty your bucket manually
aws s3 rm s3://your-bucket-name --recursive
```

#### 10. **Destroy the Stack** 💥

```bash
cdk destroy
```

We always must bootstrap the aka providion resources before we deploy out CDK app stack. So CF stack will be created called **CDKToolkit** with s3 bucket and IAM Roles.

<mark style="color:red;">Error if you haven't bootstraped into new env, so for each new environment must run:</mark> `cdk bootstrap aws://<aws_account>/<aws_region>`&#x20;

### Testing

Can do 2 types of tests:

* Fine-grained Assertions (common) – test specific aspects of the CloudFormation template (e.g., check if a resource has this property with this value)
* Snapshot Tests – test the synthesized CloudFormation template against a previously stored baseline template

:exclamation: Also we can **validate** stack build outside CDK, i.e., if you already have a CloudFormation template (perhaps exported or generated from somewhere else

```javascript
 //Template.fromStack(MyStack) -> stack built in CDK
 const template = Template.fromStack(stack);

/*Template.fromStack(MyStack) -> for stack outside CDK, like
using existing CF templates (as raw JSON or YAML strings) */
const templateString = `
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: "my-example-bucket"

  MyLambda:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: "MyLambdaFunction"
      Handler: "index.handler"
      Role: arn:aws:iam::123456789012:role/execution-role
      Code:
        S3Bucket: "my-example-bucket"
        S3Key: "lambda-code.zip"
      Runtime: "nodejs14.x"
`;

const externalTemplate = Template.fromString(templateString);

```

```javascript
// test cdk-app.test.js

const { Template } = require('aws-cdk-lib/assertions');

   const template = Template.fromStack(stack);

   template.hasResourceProperties('AWS::SQS::Queue', {
     VisibilityTimeout: 300
  });
});

```

or // Define the S3 URI where the template is stored const templateUrl = 's3://my-bucket/my-cloudformation-template.yaml';

// Load the template from the S3 bucket const externalTemplate = Template.fromUrl(templateUrl);





***

#### CloudFormation :white\_check\_mark:

CloudFormation templates describe AWS resources and their relationships. Using YAML (or JSON) we define and manage AWS as IaC.

**Key Features**:

* Defines AWS resources like EC2, S3, Lambda, etc.
* Supports parameters, conditions, mappings, and outputs.
* Allows nesting templates and using macros.

#### Instinisc (Logical) Functions &#x20;

* **Ref**: References the value of a parameter or resource. In `!Ref BucketNameParam`, it retrieves the value of the parameter `BucketNameParam`.
* **Fn::GetAtt**: Retrieves an attribute from a resource. In `!GetAtt MyBucket.Arn`, it fetches the ARN attribute of `MyBucket`.
* **Fn::FindInMap**: Retrieves a value from a `Mappings` section. In `!FindInMap [RegionMap, !Ref "AWS::Region", AMI]`, it looks up the AMI ID based on the current region.
* **Fn::ImportValue**: Imports an exported value from another stack, as seen with `!ImportValue MyExportedBucketName`.
* **Fn::Join**: Joins multiple strings with a delimiter. `!Join [ "-", ["MyApp", !Ref "AWS::Region", "Bucket"] ]` combines the strings with a hyphen (`-`).
* **Fn::Sub**: Substitutes variables inside a string. `!Sub "arn:aws:s3:::${BucketName}"` substitutes the `BucketName` parameter.
* **Fn::ForEach**: Iterates over a list (though note that `Fn::ForEach` is not natively available in CloudFormation but used in other AWS services like SAM).
* **Fn::ToJsonString**: Converts a dictionary or list into a JSON string, like `!ToJsonString {"Key1": "Value1", "Key2": "Value2"}`.
* **Fn::If, Fn::Not, Fn::Equals**: Logical functions used for conditional logic in templates. In `!If [IsProduction, "Production Value", "Non-Production Value"]`, it returns values based on the condition.
* **Fn::Base64**: Encodes a string into Base64, such as `!Base64 "Hello World"`.
* **Fn::Cidr**: Generates a list of CIDR blocks, as in `!Cidr [ "10.0.0.0/16", 8, 4 ]`.
* **Fn::GetAZs**: Retrieves availability zones for the region. `!GetAZs ""` gives all AZs in the region.
* **Fn::Select**: Selects an element from a list. `!Select [ 0, !GetAZs "" ]` selects the first AZ from the list of available AZs.
* **Fn::Split**: Splits a string into a list. `!Split [ ",", "value1,value2,value3" ]` creates a list `["value1", "value2", "value3"]`.
* **Fn::Transform**: Applies a macro, like `AWS::Include` or `AWS::Serverless-2016-10-31`, to include or transform parts of the template.
* **Fn::Length**: Returns the length of a string or list. `!Length [ "item1", "item2", "item3" ]` returns `3`.



#### SAM

**Purpose**: for building serverless applications on AWS. Extends CloudFormation.YAML (or JSON). SAM templates are essentially CloudFormation templates with additional <mark style="background-color:yellow;">**serverless-specific shorthand syntax.**</mark>&#x20;

:white\_check\_mark: SAM fully supports all the **CF built-in pseudo parameters**, such as `AWS::Region` and `AWS::AccountId, AWS::Partition, AWS::URLSuffix, AWS::NotificationARNs`

:white\_check\_mark: Fully supports all **CF** intrinsic functions. Like TABLE\_NAME: **!Ref** MyDynamoDBTable

#### :white\_check\_mark: Support for Managed **Built-in Policies**

SAM simplifies attaching permissions to resources using predefined policies:

* <mark style="background-color:yellow;">**S3ReadPolicy**</mark> -> Grants read-only permissions to an Amazon S3 bucket

```yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.9
      CodeUri: ./code
      Policies:
        - S3ReadPolicy:
            BucketName: my-example-bucket

```

* <mark style="background-color:yellow;">**SQSPollerPolicy**</mark> -> permissions to poll messages from SQS.

```yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.9
      CodeUri: ./code
      Policies:
        - SQSPollerPolicy:
            QueueName: my-example-queue

```

* <mark style="background-color:yellow;">**DynamoDBCrudPolicy**</mark>  -> full CRUD permissions (Create, Read, Update, Delete) to a specified Amazon DynamoDB table.
* **DynamoDBReadPolicy**: Grants read-only access to a DynamoDB table.
* **S3CrudPolicy**: Grants full CRUD permissions for an S3 bucket.
* **KinesisCrudPolicy**: Grants full CRUD permissions for Amazon Kinesis streams.
* **SNSTopicPolicy**: Grants permissions to publish to an Amazon SNS topic.
* **SESPolicy**: Grants permissions for sending email with Amazon SES.
* **StepFunctionsExecutionPolicy**: Grants permissions to start executions of a Step Functions state machine.



**Key Features**:

* Adds simplified definitions for serverless resources ( SAM automatically expands **shorthand syntax into full CloudFormation resource** definitions during deployment.)
  * `Transform: AWS::Serverless-2016-10-31` -> declares the SAM template and enables SAM features.
  * `AWS::Serverless::Function` -> defines Lambd&#x61;**.** Automatically handles IAM roles, event sources, and deployment configuration
  * SAM provides easy-to-use shorthand for **common Lambda triggers** under the `Events` property of `AWS::Serverless::Function`
  * `AWS::Serverless::LayerVersion` -> defining Lambda Layer versions T OSIMPLIFY reusing dependancies and shared libraries.
  * `AWS::Serverless::Api` -> defines an Amazon API Gateway REST API. Build-in support for CORS, Swagger, and stages, OpenAPI definitions and inline routes/events.
  * `AWS::Serverless::SimpleTable` -> defined DynamoDB tables with minimal configuration
  * AWS::Serverless::StateMachine -> Step function State machine



#### Lambda

```yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.9
      CodeUri: ./code
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /hello
            Method: get
            
  # Layers          
  Resources:
  MyLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      LayerName: MySharedLayer
      ContentUri: ./layer-code
      CompatibleRuntimes:
        - python3.9


```

#### Lambda Triggers

```yaml
#s3
Events:
  S3Event:
    Type: S3
    Properties:
      Bucket: MyBucket
      Events: s3:ObjectCreated:*
      
#db
Events:
  DynamoEvent:
    Type: DynamoDB
    Properties:
      Stream: !GetAtt MyTable.StreamArn
#SQS
Events:
  SqsEvent:
    Type: SQS
    Properties:
      Queue: !Ref MyQueue
      
#SNS
Events:
  SnsEvent:
    Type: SNS
    Properties:
      Topic: !Ref MySnsTopic



```



#### API

```yaml
Resources:
  MyApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: dev
      DefinitionBody:
        swagger: "2.0"
        paths:
          /hello:
            get:
              x-amazon-apigateway-integration:
                httpMethod: POST
                uri: !Sub "arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${MyFunction.Arn}/invocations"

```

#### DaynamoDB

```yaml
Resources:
  MyTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey:
        Name: id
        Type: String
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5

```





