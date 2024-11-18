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

:exclamation: <mark style="color:blue;">**Resources**</mark><mark style="color:blue;">: (Required)</mark> The AWS resources you want to create (e.g., EC2 instances, S3 buckets).

* Declared and can reference each other

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

**`service-provider::service-name::data-type-name`** = AWS::S3::Bucket or AWS::EC2::Instance. All Refs Link:[https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

<mark style="color:blue;">**AWSTemplateFormatVersion**</mark><mark style="color:blue;">: (</mark>Optional) The version of the CloudFormation template format.

<mark style="color:blue;">**Description**</mark><mark style="color:blue;">:</mark> (Optional) A brief description of the template's purpose.

<mark style="color:blue;">**Parameters**</mark>: (Optional) User-defined input values for the template (e.g., instance size, key pairs, DB username).&#x20;

* Useful if you need to reuse them for div env or users
* Leverage **!Ref** function to reference all parameters
* Include constrains and validations like RegEx and default value
* If the resource configuration is likely to change -> make it a parameter :white\_check\_mark: **so you won't nee to update an existing template**

**Outputs**: (Optional) Values to be returned after the stack is created, like the public IP of an EC2 instance.

**Mappings**: (Optional) A way to define static data like region-specific AMI IDs.

**Conditions**: (Optional) Logic to control resource creation based on conditions (e.g., create resources only in certain regions).

**Transform**: (Optional) Used if you're using macros or AWS-specific functions like AWS::Include for Lambda.

**Metadata**: (Optional) Extra information or descriptive data about the template.

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



