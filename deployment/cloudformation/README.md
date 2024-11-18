---
icon: gift
---

# CloudFormation

Service that models and helps set up resources, you create a template that **describes all the AWS resources that you want** (like Amazon EC2 instances or Amazon RDS DB instances), and CloudFormation provisions and configures those. On stack delete all underlying resources are deleted.

## Benefits

* **All Infrastructure as code (IaC):** simplifies management and allows to quickly replicate infrastructure. By reusing  CloudFormation template to create your resources in a consistent and repeatable manner.
* No resources are manually created, which is excellent for control, and no need to configure them to work together.
* A **template** describes all your resources and their properties. Template describes all the AWS resources that you want (like ElasticIP, EC2 instances, Security Groups, RDS DB instances), and CloudFormation takes care of provisioning and configuring those resources for you
* CloudFormation is Free, you only pay for services they used when they were creating the resource. You can estimates the costs of your resources using the cloud formation template.
* Cost - saving strategy for  development or test environment, you can enable the auto deletion of templates at 5 pm and recreate at 8 am safely, to avoid resources running overnight.
* **Productivity:** with declarative programming → no need to figure out ordering and orchestration, I need to create a DB first or instance… the template a smart enough to figure it out.
* Leverage exisitng templates.
* Easily control and track changes to infrastructure by simply tracking diff in templates, as they are text files, easy to track changes to your infrastructure.
* It's a common language to describe and provision all the infrastructure resources in your cloud environment.

### Infrastructure composer

## Templates

When you use CloudFormation, you work with **templates** and **stacks**. CloudFormation uses these templates as blueprints for building your AWS resources.

### Template Sections

Every CloudFormation template consists of one or more sections, each serving a specific purpose.

#### <mark style="color:blue;">**AWSTemplateFormatVersion**</mark>

The version of the CloudFormation template format.

#### <mark style="color:blue;">**Description**</mark>&#x20;

A brief description of the template's purpose.

#### <mark style="color:blue;">**Parameters**</mark> &#x20;

User-defined input values for the template (e.g., instance size, key pairs, DB username)

#### :red\_circle: <mark style="color:blue;">**Resources**</mark><mark style="color:blue;">: (Required)</mark>&#x20;

The AWS resources you want to create (e.g., EC2 instances, S3 buckets).

```yaml
Resources:
  <ResourceLogicalName>:
    Type: <ResourceType>
    Properties:
      <Property1>: <Value1>
      <Property2>: <Value2>
      ...
      #results into 
Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-bucket-name

```

**`service-provider::service-name::data-type-name`** = AWS::S3::Bucket or AWS::EC2::Instance. All Refs Link:[https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)&#x20;

* Useful if you need to reuse them for div env or users
* Leverage **!Ref** function to reference all parameters
* Include constrains and validations like RegEx and default value
* If the resource configuration is likely to change -> make it a parameter :white\_check\_mark: **so you won't nee to update an existing template**

#### <mark style="color:blue;">**Outputs**</mark>

&#x20;Values to be returned after the stack is created, like the public IP of an EC2 instance.

* **Expose Critical Values**: Display details like resource names, IDs, or endpoints.
  * The public URL of a deployed web server.
* **Ease of Access**: Quickly retrieve key information via the CloudFormation console, CLI, or SDK.

```yaml
Outputs:
  WebsiteURL:
    Description: "URL for the application"
    Value: !Join ['', ['http://', !GetAtt MyWebServer.PublicIp]]
    Export:
      Name: WebsiteURL

```

```yaml
Outputs:
  WebsiteURL:
    Value: !Join
      - ''
      - - http://
        - !GetAtt WebServer.PublicDnsName
        - /wordpress
    Description: WordPress Website
    # if public DNS name:ec2-123-45-67-89.compute-1.amazonaws.com.
    # result is http://ec2-123-45-67-89.compute-1.amazonaws.com/wordpress
```

* Delimiter (`''`): Nothing is placed between the elements being joined.
* Elements:
  * `http://`: This is the protocol part of the URL.
  * `!GetAtt WebServer.PublicDnsName`: This fetches the public DNS name of the EC2 instance (`WebServer`).
  * `/wordpress`: This is the path to the WordPress application.
* **Pass Information to Another Stack**: Outputs can export values that other CloudFormation stacks can reference.
  * Exporting a VPC ID for use in a dependent stack.

```yaml
#This stack creates a VPC and exports its ID for use by other stacks.

Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

Outputs:
  VPCID:
    Description: "The ID of the VPC"
    Value: !Ref MyVPC
    Export:
    #Ensure exported names are unique within the AWS account and region.
      Name: MyVPCID 
```

Consumer Stack

```yaml
#This stack imports the VPC ID from NetworkStack and uses it to create a subnet.


Parameters:
  SubnetCIDR:
    Type: String
    Default: 10.0.1.0/24

Resources:
  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !ImportValue MyVPCID
      CidrBlock: !Ref SubnetCIDR
```

<mark style="color:red;">**CloudFormation Error**</mark><mark style="color:red;">: If another stack (dependent stack) is using an</mark> <mark style="color:red;"></mark><mark style="color:red;">`!ImportValue`</mark> <mark style="color:red;"></mark><mark style="color:red;">from the source stack, CloudFormation will</mark> <mark style="color:red;"></mark><mark style="color:red;">**not let you delete the source stack**</mark><mark style="color:red;">.</mark>

#### <mark style="color:blue;">**Mappings**</mark>

&#x20;A way to define static data like region-specific AMI IDs, to differentiate between different env, regions. All hardcoded in the template. **AMIs are region-specific.**&#x20;

**Mappings** are perfect for **static, predefined values** that don't change during runtime

```yaml
#Simple mapping for ami
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0abcdef1234567890
      InstanceType: t2.micro
    eu-west-1:
      AMI: ami-0987654321abcdef0
      InstanceType: t3.micro

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
    #!FindInMap: Looks up the AMI ID for the current region (!Ref AWS::Region).
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
       InstanceType: !FindInMap [RegionMap, !Ref "AWS::Region", InstanceType]

```

```yaml
# mapping 
Parameters:
  Environment:
    Type: String
    Default: Dev
    AllowedValues:
      - Dev
      - Prod
    Description: Choose the environment (Dev or Prod).

Mappings:
  EnvRegionMap:
    Dev:
      us-east-1: ami-0abcdef1234567890
      eu-west-1: ami-0987654321abcdef0
    Prod:
      us-east-1: ami-0123456789abcdef0
      eu-west-1: ami-0fedcba9876543210

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
    #!FindInMap: Resolves the AMI ID based on both the environment and region
    #If the stack is in Prod and region us-east-1, the AMI will be ami-0123456789abcdef0.

      ImageId: !FindInMap [EnvRegionMap, !Ref Environment, !Ref "AWS::Region"]
      InstanceType: t2.micro

```

Most common **pseudo-parameters** in CloudFormation:

1. **`AWS::Region`**
   * Represents the region in which the stack is being deployed.
   *   Example:

       ```yaml
       yamlCopy codeImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
       ```
2. **`AWS::AccountId`**
   * The AWS account ID of the account deploying the stack.
   *   Example:

       ```yaml
       yamlCopy codeBucketName: my-app-bucket-!Ref "AWS::AccountId"
       ```
3. **`AWS::StackName`**
   * The name of the stack as provided during creation.
   *   Example:

       ```yaml
       yamlCopy codeTag:
         Key: Stack
         Value: !Ref "AWS::StackName"
       ```

#### <mark style="color:blue;">**Conditions**</mark>

&#x20;Logic to control resource creation based on conditions (e.g., create resources only in certain regions, env like dev/ prod). They help make your templates dynamic, enabling conditional logic.

```yaml
# Useful to selectively deploy resources based on the environment
# E.g. to provision a larger EC2 instance (EnhancedInstance) for produ, but not dev
Parameters:
  Environment:
    Type: String
    AllowedValues: 
      - dev
      - prod

Conditions:
  #Evaluates to true if the Environment parameter is set to prod.
  IsProd: !Equals [!Ref Environment, prod]
  # Returns true if the Environment is not dev.
  IsNotDev: !Not [!Equals [!Ref Environment, dev]]
  #!And: A logical function that evaluates multiple conditions. 
  # It returns TRUE ONLY if ALL conditions inside are true
  UseEnhancedInstance: !And
    - !Equals [!Ref Environment, prod]
    - !Condition IsNotDev

Resources:
  EnhancedInstance:
    Type: AWS::EC2::Instance
    Condition: UseEnhancedInstance
    Properties:
      InstanceType: t3.large
  StandardInstance:
    Type: AWS::EC2::Instance
    Condition: !Not [UseEnhancedInstance]
    Properties:
      InstanceType: t3.micro
      
      
```

**Key Intrinsic Functions for Conditions**

1.  **`!Equals` \[value1, value2]**

    * Compares two values, like `MyParam` parameter with the value `prod`.

    &#x20;`!Equals [!Ref MyParam, prod]`
2.  **`!Not [value]`**

    * Negates a condition or comparison, here `!Not` **negates** the result of the `!Equals` so if the environment is not `prod`, it returns `true`.

    `NotProd: !Not [!Equals [!Ref Environment, prod]]`
3.  **`!And [condition1, condition2]`**

    * Evaluates multiple conditions and returns `true` if all are `true`. If the env is prod and region.

    &#x20;`IsProdAndDev: !And - !Equals [!Ref Env, prod] - !Equals [!Ref Region, us-west-1]`
4.  **`!Or [condition1, condition2]`**

    * Evaluates multiple conditions and returns `true` if at least one (either) is `true`

    `!Or [!Equals [!Ref Env, dev], !Equals [!Ref Env, test]]`
5.  **`!If`\[ConditionName, ValueIfTrue, ValueIfFalse]**

    * Used to evaluate and apply conditions within resource properties.

    `InstanceType: !If [UseEnhancedInstance, t3.large, t3.micro]`

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues:
      - dev
      - prod
      - staging

Conditions:
  IsProd: !Equals [!Ref Environment, prod]
  IsDev: !Equals [!Ref Environment, dev]

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
# to choose instance type
      InstanceType: !If [IsProd, t3.large, !If [IsDev, t3.micro, t3.medium]]
#if IsProd is TRUE  the InstanceType is set to t3.large.
# If IsProd is FASLE, it then evaluates the SECOND !If statement:so If IsDev is TRUE
# the InstanceType is set to t3.micro but If IsDev is FASLE (i.e., the env is staging)
# the InstanceType is set to t3.medium.
```



**Transform**

&#x20;Used if you're using macros or AWS-specific functions like AWS::Include for Lambda.

**Metadata**

&#x20;Extra information or descriptive data about the template.

### CloudFormation template formats <a href="#template-formats" id="template-formats"></a>

CloudFormation templates are divided into different sections, and each section is designed to hold a specific type of information. Some sections must be declared in a specific order, and for others, the order doesn't matter. However, as you build your template, it can be helpful to use the logical order shown in the following examples because values in one section might refer to values from a previous section.

#### JSON

<details>

<summary>JSON-formatted template with all available sections.</summary>

```json
{
  "AWSTemplateFormatVersion" : "version date",

  "Description" : "JSON string",

  "Metadata" : {
    template metadata
  },

  "Parameters" : {
    set of parameters
  },
  
  "Rules" : {
    set of rules
  },

  "Mappings" : {
    set of mappings
  },

  "Conditions" : {
    set of conditions
  },

  "Transform" : {
    set of transforms
  },

  "Resources" : {
    set of resources
  },
  
  "Outputs" : {
    set of outputs
  }
}
```



</details>

* In JSON-formatted templates, comments are not supported. JSON, by design, doesn't include a syntax for comments, which means you can't add comments directly within the JSON structure, but to include explanatory notes / documentation by adding **metadata**

#### YAML :heart\_eyes::heart\_hands:

<details>

<summary>YAML-formatted template with all available sections.</summary>

```yaml
---
AWSTemplateFormatVersion: version date

Description:
  String

Metadata:
  template metadata

Parameters:
  set of parameters

Rules:
  set of rules

Mappings:
  set of mappings

Conditions:
  set of conditions

Transform:
  set of transforms

Resources:
  set of resources

Outputs:
  set of outputs

```



</details>

* YAML supports inline comments by using the `#` symbol.



#### Useful Links

[https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/gettingstarted.walkthrough.html#gettingstarted.walkthrough.viewapp](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/gettingstarted.walkthrough.html#gettingstarted.walkthrough.viewapp)

[https://docs.aws.amazon.com/cli/latest/reference/cloudformation/#cli-aws-cloudformation](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/#cli-aws-cloudformation)



