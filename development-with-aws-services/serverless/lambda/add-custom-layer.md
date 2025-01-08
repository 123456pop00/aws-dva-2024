---
icon: layer-plus
description: '-> Lambda layers must be uploaded in a zip format'
---

# Add custom layer

* **Lambda Layers**:  are immutable once created, a layer version cannot be modified.
* Can be created with CDK but will be **destroyed (cdk destroy)** &  :no\_entry: :no\_entry: :no\_entry: doesn't provide a native retain policy for Lambda specifying a "retain"&#x20;
* `removalPolicy` can't be set for Lambda or Layers i.e. like for S3 or DDB
  * `const bucket = new s3.Bucket(this, "MyBucket", { removalPolicy: cdk.RemovalPolicy.RETAIN, });`
* After creating a layer, you can export its ARN (via `cdk.CfnOutput`) and use it in future deployments.
* Detach the Lambda Layer or Function from the CDK Stack / Manually / Separate Stack

## Adding layer for nodejs aws-sdk 📟 CloudShell

```bash
cd cloudshell-user/
mkdir -p aws-sdk-layer/nodejs
cd aws-sdk-layer/nodejs/
npm install aws-sdk
zip -r ../package.zip .

//go up & Publish a Lambda layer version 
cd ..
aws lambda publish-layer-version \
  --region eu-central-1 \
  --layer-name nodesdk \
  --description "Custom nodejs layer for sdk" \
  --zip-file fileb://package.zip \
  --compatible-runtimes nodejs22.x




```

### This should work with added layer

```javascript

import AWS from 'aws-sdk';

const dynamodb = new AWS.DynamoDB.DocumentClient();
const rekognition = new AWS.Rekognition();
const apigateway = new AWS.APIGateway();
const dynamoStreams = new AWS.DynamoDBStreams();
...
```





