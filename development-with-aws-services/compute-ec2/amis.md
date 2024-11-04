# AMIs

**Customisation and Replication**:\
AMIs encapsulate the configured state of an EC2 instance, enabling <mark style="color:red;">quick replication and deployment of identical instances.</mark> This allows for the creation of ready-to-use custom instances, such as a preconfigured Node.js + Express server, with immediate setup.

**Versioning & Updates**:\
Supports versioning to manage changes and updates over time, ensuring consistency across deployments. This helps track changes and roll back if necessary.

**Launch Options**:\
You can launch EC2 instances from:

* **Public AMIs** provided by AWS.
* **Custom AMIs** that you create and maintain.
* **AWS Marketplace** AMIs for pre-built solutions.

**Benefits**:\
AMIs allow for faster boot times since all required software is **prepackaged**, reducing setup time.

**Golden AMI**: to redeploy instances in new regions. Ensure that other **dependent** resources in your environment are also transferred to maintain functionality.

**EBS Snapshot Storage**: Each AMI has a storage cost based on the size of the underlying snapshots. AWS charges per GB per month for EBS snapshot storage.

**No Direct  S3 - EC2 Launch** :rocket: :no\_entry:: EC2 instances cannot launch directly from S3-stored AMI files; they must be re-imported as AMIs first.
