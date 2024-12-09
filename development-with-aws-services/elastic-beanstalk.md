---
icon: coffee-bean
cover: >-
  https://images.unsplash.com/photo-1550859492-d5da9d8e45f3?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw3fHxjb2xvcnN8ZW58MHx8fHwxNzMzNjUzNTEzfDA&ixlib=rb-4.0.3&q=85
coverY: 0
---

# Elastic Beanstalk

## Features

* AWS Elastic Beanstalk supports Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker
* It AWS CloudFormation to launch the resources in your environment and propagate configuration changes
* Can deploy web or worker
  * **Web Applications**:
    *   **Single-instance environment**: Great for dev/testing or low-traffic apps.

        * Still has ASG with capacity **Min/Max** Set to `1`. <mark style="background-color:orange;">If the instance fails, the ASG replaces it.</mark>
        *   To ass TLS  to your Beanstalk web app for HTTPS  do:&#x20;

            **a) Attach to Load Balancer**: Go to the **EC2 Console** → Load Balancers -> Find the LB for your BS env -> Under **Listeners**, add or edit an HTTPS listener ->Select your TLS certificate and save.
        * **b)** Create a `.config` file in `/ebextensions` to define HTTPS rules and attach a certificate programmatically.

        &#x20;
  * **High Availability (HA) with ALB + ASG** : Scales automatically with multiple instances for production.
  * **Worker Applications**:
    * Handles background jobs.
    * Processes tasks from an Amazon SQS queue. Requests are in a SQS queue and the EC2 instances pull the messages to process them. Scaling depends on the number of SQS messages in the queue.
* To deploy a worker application that processes periodic background tasks, the application bundle must include a **cron.yaml file.**
* Beanstalk provides a **lifecycle policy** to manage application **versions** effectively, you can remove older application versions based on specific criteria:
  * **Time-based**: you can set a policy to discard versions that are older than a certain duration (e.g., 180 **days**).
  * **Count-based**:  limit the total number of application versions stored (e.g., to a maximum of 200).
  * Elastic Beanstalk can store a maximum of 1000 application versions. If older versions are not removed, you may be unable to deploy new iterations of your application.
  * Versions currently in use by your environments will not be deleted, r**egardless of the lifecycle policy.**
* To deploy a new version of the application, package your application as a zip or war file and deploy it using eb deploy command.
* To configure resources & customise EB **environment** -> Config files in `.ebextensions/` directory in the **root** of the source code
  * Provides more advanced, repeatable, and version-controlled customization option
  * Runs during the EB deployment lifecycle, applies changes automatically.
  * All files have `.config` extension and should be in <mark style="background-color:orange;">YAML(preferred)</mark> or JSON format
* &#x20;`.ebextensions` directory allows you to manage environment settings that **cannot be configured through the Elastic Beanstalk console** like RDS (Relational Database Service), Elasticache, and DynamoDB can also be specified within these extensions.
* **Environment configurations** like VPC, RDS, and load balancers are specific to the account and region
  * RDS can be created inside (dev env) and outside the EB env ( prod)
* Manual migration for cross-account -> EB doesn't have direct export/import
*   Manual migration for LB, **we can't to directly change the load balancer type** from the console.&#x20;

    * **Migrate DNS settings** (e.g., Route 53)  to the new environment's load balancer
      * **Old Environment**: Your app's domain (e.g., `www.example.com`) points to the old EB environment's URL (e.g., `old-env.elasticbeanstalk.com`).
      * **New Environment**: After you deploy to the new EB environment with the desired load balancer (ALB), you'll update your DNS settings.
      * **CNAME Swap**: You update the CNAME record for your domain to point to the new EB environment's URL (e.g., `new-env.elasticbeanstalk.com`).




*   When you **clone an environment:**

    * ✅ **Resources are replicated**: Elastic Beanstalk will create a copy of all resources associated with the original environment, such as:
      * EC2 instances
      * ALB or ELB
      * Auto Scaling Group
      * Security Groups
      * Configuration settings
    * ✅ **RDS instance is cloned**, but...
      * ⚠️ **RDS data is not copied**: The new RDS instance starts with a clean slate. If you need the same data, you must manually back it up and restore it to the cloned instance.
    * :exclamation:Does **not copy application data** or any configuration outside of **that** environment.


* Cloning can also be a good starting point for **Blue/Green deployments for isolated features**
* Via **CLI BS** supports `--option-settings` to specify the **custom AMI ID** (`ami-0123456789abcdef0`)
  * **Custom AMI Compatibility**: The AMI you use should be compatible with the selected platform (e.g., Amazon Linux 2, Node.js, etc.).
  * **Elastic Beanstalk Managed Updates**: Once you use a custom AMI, Beanstalk no longer manages updates or patches to the AMI, so you need to manage it yourself.
  *   **Scaling Configuration**: If you’re using a custom AMI, ensure that the AMI has the right configurations for scaling and management (e.g., proper permissions, roles, etc.).





## Deployment Strategies&#x20;

{% tabs %}
{% tab title="All At Once" %}
* Deploys the new version to all instances simultaneously
* &#x20;It is the quickest strategy but has some **downtime as we deploy to existing instance(s)**
* Works in single instance and LB env
* Cheapest , fastest & **default deployment policy**
* Rollback  -> manual redeploy
{% endtab %}

{% tab title="Rolling" %}
* Doesn't work in single instance env
* Attaches and detaches instances from ALB in batches, taking specified size **out of service while deploying new version on them &** monitoring their health before proceeding to the next batch, we deploy to new and existing instances.
  * **Batch size** defines how many instances are updated at once during the deployment.
  * Can be set as **percentage** or **fixed number** of instances.  You need to explicitly define the **BatchSizeType** to choose between the two.
* Minimizes downtime and allows you to have instances running both the old and new versions during the deployment process
* Reduced capacity\* so you accept the risk of lower level of capacity as servers are taken down
* No additional cost
* Rollback  -> manual redeploy

```yaml
option_settings:
  aws:elasticbeanstalk:command:
    DeploymentPolicy: Rolling                # Rolling update policy
    RollingUpdateBatchSize: 50               # 50 instances or 50% based on the BatchSizeType
    RollingUpdateBatchSizeType: Percentage   # Can also be "Fixed" for fixed instances
    RollingUpdateEnabled: true               # Enable rolling updates
    MaxBatchSize: 2                          # Optional max batch size, a fixed number
    PauseTime: 10                            # Pause time in seconds between batches

```

If a deployment fails after one or more batches completed successfully, the completed batches run the new version of your application while any pending batches continue to run the old version. You can identify the version running on the instances in your environment on the health page in the console.

* If you **terminate instances from the failed deployment,** Elastic Beanstalk replaces them with instances running the application version from the **most recent successful deployment.**
{% endtab %}

{% tab title="Rolling + Additional Batches" %}
* Doesn't work in single instance env
* **Additional batches** make sure that new instances are added before removing the old ones. This ensures we have **no loss of capacity.** This means that new instances (with the updated application version) are created and launched first, before terminating the old instances. This allows you to maintain full capacity during the deployment.
* Rollback  -> manual redeploy

```yaml
option_settings:
  aws:elasticbeanstalk:command:
    DeploymentPolicy: "RollingWithAdditionalBatches"  # Sets deployment type
    BatchSizeType: "Percentage"  # Or "Fixed" depending on what you prefer
    BatchSize: 50  # If percentage, 50 means 50% of instances at a time. Or set a fixed number.
    MaxBatchSize: 2  # For percentage-based, dafelty-limit this limits the number of instances to deploy in the batch.

```


{% endtab %}

{% tab title="Immutable" %}
* When you deploy a new version using **Immutable**, Elastic Beanstalk**k** creates a new environment (with its own resources, like a new ALB and EC2 instances). This new environment is fully set up and tested before it gets traffic.It gets it from swapping the CNAME (its the actual process of updating the DNS record to point to the new environment's load balance).
* Elastic Beanstalk creates a new, fully separate environment for the new version, similar to the <mark style="background-color:green;">**Green**</mark> environment in Blue/Green.
* The old environment stays intact until the deployment is successful.
* The old environment **remains intact** but is no longer receiving traffic after the CNAME swap. It's still there, but it's in a **"standby"** state, and you can choose to delete it manually later
* **Zero downtime:** New environment is completely isolated until it's ready to replace the old one.
* **Safest - we duplicate infrastructure** ( mission-critical apps)
* Rollback -> swap the CNAME back to the old environment
* Can be slower since we provisioning entire duplicate infrastructure than rolling updates there but guarantees no interruption&#x20;
{% endtab %}

{% tab title="Traffic Splitting " %}
* Doesn't work in single instance env
* We point a % of traffic to the new env and after a period of time and validation switch over
* It is effectively a **canary deployment** mechanism where traffic is gradually split between multiple versions of the application, and you can adjust the allocation based on performance
* Allows for **gradual and controlled rollout of new application versions**
* Deployment :hourglass: can depend on Canary Timeout

```yaml
option_settings:
  aws:elasticbeanstalk:environment:
    TrafficSplittingEnabled: true   # Enable traffic splitting
    TrafficSplittingPercentage: 30  # Route 30% of traffic to the new version

```


{% endtab %}

{% tab title="Blue/Green" %}
_Not direct feature of EB_

* You have two environments: **Blue** (current version) and **Green** (new version).
* The **Blue** environment serves live traffic, while you deploy the **Green** environment with the new version.
* After the **Green** environment is fully deployed and tested, you switch traffic to **Green** by updating the DNS (CNAME swap). The traffic switch happens **all at once** after the green environment is validated
* Once traffic is shifted to **Green**, the **Blue** environment can be deleted or kept as a **backup**.
* DNS change happens at the ALB level
* DB must be outside the EB env because when the env is terminated we lose al resources
{% endtab %}
{% endtabs %}

*   Some policies replace all instances during the deployment or update. This causes all accumulated Amazon EC2 burst balances to be lost. It happens in the following cases:

    1. Managed platform updates with instance replacement enabled
    2. Immutable updates
    3. Deployments with immutable updates or traffic splitting enabled

    \






#### Useful CLI

`aws elasticbeanstalk describe-environments`

`aws elasticbeanstalk delete-application --application-name`` `_`name-of-the -env`_` ``--terminate-env-by-force`

`aws elasticbeanstalk describe-environments --query "Environments[?Status!='Terminated']"`





### Deploying with CodePipeline&#x20;

On set up create [AWSCodePipelineServiceRole-eu-north](https://us-east-1.console.aws.amazon.com/iam/home?region=eu-north-1#/policies/details/arn%3Aaws%3Aiam%3A%3A060683702247%3Apolicy%2Fservice-role%2FAWSCodePipelineServiceRole-eu-north-1-myFirstPipeline)-mypipeline service Role and ensure it has the&#x20;

AWSCodePipelineServiceRole-eu-north... Should Have&#x20;

<figure><img src="../.gitbook/assets/iamServiceRoleforBS.png" alt=""><figcaption></figcaption></figure>

<mark style="color:red;background-color:red;">**Including\***</mark>

`{ "Effect": "Allow", "Action": "logs:PutRetentionPolicy", "Resource": "arn:aws:logs:eu-north-1:<ACCOUNT_ID>:log-group:/aws/elasticbeanstalk/*" }`

