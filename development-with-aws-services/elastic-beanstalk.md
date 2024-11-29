# Elastic Beanstalk

#### Useful CLI

`aws elasticbeanstalk describe-environments`

`aws elasticbeanstalk delete-application --application-name`` `_`name-of-the -env`_` ``--terminate-env-by-force`

`aws elasticbeanstalk describe-environments --query "Environments[?Status!='Terminated']"`





#### Deploying with CodePipeline&#x20;

On set up create [AWSCodePipelineServiceRole-eu-north](https://us-east-1.console.aws.amazon.com/iam/home?region=eu-north-1#/policies/details/arn%3Aaws%3Aiam%3A%3A060683702247%3Apolicy%2Fservice-role%2FAWSCodePipelineServiceRole-eu-north-1-myFirstPipeline)-mypipeline service Role and ensure it has the&#x20;

AWSCodePipelineServiceRole-eu-north... Should Have&#x20;

<figure><img src="../.gitbook/assets/iamServiceRoleforBS.png" alt=""><figcaption></figcaption></figure>

<mark style="color:red;background-color:red;">**Including\***</mark>

`{ "Effect": "Allow", "Action": "logs:PutRetentionPolicy", "Resource": "arn:aws:logs:eu-north-1:060683702247:log-group:/aws/elasticbeanstalk/*" }`

