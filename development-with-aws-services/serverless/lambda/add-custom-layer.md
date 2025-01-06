---
icon: layer-plus
---

# Add custom layer

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





