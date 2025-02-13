## Takeaways from Practice Exams ##

- `AWS PrivateLink` enables private connectivity between VPCs and supported AWS or third-party services, ensuring that traffic does not traverse the public internet.
- `AWS VPN` connections are typically used for secure communication between on-premises environments and AWS
- `AWS Batch` automatically manages the underlying infrastructure, scales based on workload and simplifies batch job management.
- `AWS Application Migration Service` enables lift-and-shift migrations by replicating VMs from the on-premises data center to AWS
    - `AWS Sever Migration Service` has been deprecated in favor of AWS Application Migration Service, which provides a more streamlined and modern approach to lift-and-shift migrations.
- A `custom origin` can point to an on-premises server and `CloudFront` is able to cache content for dynamic websites. `CloudFront` can provide performance optimizations for custom origins even if they are running on on-premises servers. 
- `Amazon Elastic File System (Amazon EFS)` provides a simple, scalable, fully managed elastic NFS file system for use with AWS Cloud services and on-premises resources. It is built to scale on-demand to petabytes without disrupting applications, growing and shrinking automatically as you add and remove files, eliminating the need to provision and manage capacity to accommodate growth.
- `AWS Control Tower` simplifies account setup with built-in security guardrails. It minimizes operational overhead by automating VPC sharing and guardrail enforcement through AWS RAM.
- `Amazon FSx for Windows File Server` provides fully managed, highly reliable file storage that is accessible over the industry-standard Server Message Block (SMB) protocol. `Amazon FSx` is built on Windows Server and provides a rich set of administrative features that include end-user file restore, user quotas, and `Access Control Lists (ACLs)`.Additionally, `Amazon FSX for Windows File Server` supports `Distributed File System Replication (DFSR)` in `Single-AZ` deployments as can be seen in the feature comparison table below.
    - Uses `NTFS`
    - `AWS EFS` is only for linux
- `AWS Gateway Load Balancer` is specifically designed to simplify the deployment of security appliances. Using `GWLB` endpoints in service accounts ensures efficient routing and centralized inspection of traffic.
- `AWS physical Snowball Edge` device will provide much more inbuilt compute and storage compared to the current team’s laptops. This negates the need to rely on a stable connection to process any images and solves the team's problems easily and efficiently.
- `AWS EventBridge` is not suitable for handling high-frequency batch image processing tasks
- data transfer between `EC2` instances in the same `AZ` is **free**, while transferring data across `AZs` incurs additional charges. Keeping all instances in the same `AZ` minimizes data transfer costs while maintaining efficient processing.
- `Amazon EFS` provides scalable file storage for use with Amazon `EC2`. You can use an EFS file system as a common data source for workloads and applications running on multiple instances. The `EC2` instances can run in multiple `AZs` within a `Region` and the `NFS protocol` is used to mount the file system.
    - With `EFS` you can create mount targets in each AZ for lower latency. The application instances in each AZ will mount the file system using the local mount target.
    -
- `IAM Roles Anywhere` provides a secure and scalable method for on-premises workloads to obtain **temporary** AWS credentials. It avoids the use of long-term credentials and integrates with `IAM Identity Center `to ensure least privilege access to the `S3` bucket.

- `Service Control Policies (SCP)` are used to enforce policies across accounts in an organization
- `Tag policies` enforce tagging standards for AWS resources. By attaching the tag policy to the OU, you ensure that all `EC2` instances follow the defined tagging requirements.
- `AWS Config` cannot prevent resource creation or modification. It can detect noncompliance but requires additional tools like `Lambda` for remediation, which introduces operational overhead.
- `Amazon Aurora Global Database` is designed for globally distributed applications, allowing a single `Amazon Aurora database` to span multiple `AWS regions`. It replicates your data with no impact on database performance, enables fast local reads with low latency in each region, and provides disaster recovery from region-wide outages.
- `RedShift` is a data warehouse and used for running analytics queries on data that is exported from transactional database systems
- `Amazon DynamoDB `and `Amazon S3 `support gateway endpoints, not interface endpoints. With a gateway endpoint you create the endpoint in the `VPC`, attach a policy allowing access to the service, and then specify the route table to create a route table entry in.
- `Amazon Kinesis` enables you to ingest, buffer, and process streaming data **in real-time.** `Kinesis` can handle any amount of streaming data and process data from hundreds of thousands of sources with very low latencies. This is an ideal solution for data ingestion.
- `Amazon Simple Queue Service (SQS)` is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications. `SQS` eliminates the complexity and overhead associated with managing and operating message-oriented middleware and empowers developers to focus on differentiating work.

- `AWS Resource Access Manager (RAM)` is a service that enables you to easily and securely share AWS resources with any AWS account or within your `AWS Organization`. You can share AWS Transit Gateways, Subnets, AWS License Manager configurations, and `Amazon Route 53 Resolver` rules resources with RAM.


| Gateway Type          | Protocol(s) Supported | Client File System (Typical) | Use Case                                                                                                    |
|-----------------------|----------------------|-----------------------------|-------------------------------------------------------------------------------------------------------------|
| File Gateway          | NFS, SMB             | NTFS (Windows via SMB) / Any POSIX FS (Linux via NFS) | Stores files as objects in S3, providing a file-based interface for applications.                             |
| Volume Gateway        | iSCSI                | Any FS supported by the OS    | Offers block storage volumes backed by S3 or EBS, suitable for high-performance, low-latency applications. |
| Tape Gateway          | iSCSI-VTL            | N/A (Emulates a tape library) | Simplifies tape-based backups to AWS, providing a cost-effective solution for long-term archival.           |
| Amazon FSx File Gateway | SMB                  | NTFS                         | Provides a local cache for frequently accessed files on Amazon FSx for Windows File Server.                 |          |


- `Amazon Kendra` an intelligent search service.
- `Amazon Polly` a cloud service that converts text into speech
- Can use `AppSync` with DynamoDB to make it easy for you to build collaborative apps that keep shared data updated in real-time
    - GraphQL Api
    - `AppSync pipeline resolvers` offer an elegant server-side solution to address the common challenge faced in web applications—aggregating data from multiple database tables. 
- `Amazon AppFlow` is simply an integration service for transferring data securely between Software-as-a-Service (SaaS) applications like Salesforce, SAP, Zendesk, Slack, ServiceNow, and AWS services.
- `AWS Backup` is a centralized backup service that makes it easy and cost-effective for you to backup your application data across AWS services in the AWS Cloud, helping you meet your business and regulatory backup compliance requirements

- A `subnet` can only be associated with one `route table` at a time, but you can associate multiple subnets with the same subnet `route table`. You can optionally associate a `route table` with an `internet gateway` or a `virtual private gateway` (gateway `route table`). This enables you to specify routing rules for inbound traffic that enters your `VPC` through the `gateway`

- stopping – The instance is preparing to be stopped. Take note that you will not billed if it is preparing to stop however, **you will still be billed if it is just preparing to hibernate.**



- `AWS Transit Gateway` provides a hub and spoke design for connecting VPCs and on-premises networks.


-  Keep in mind that an `EC2 instance` has an underlying physical host computer. If the instance is stopped, `AWS` usually moves the instance to a new host computer. Your instance may stay on the same host computer if there are no problems with the host computer

- You can’t have an `Amazon S3 managed encryption key` for client-side encryption. As its name implies, an `Amazon S3 managed key` is fully managed by AWS and also rotates the key automatically without any manual intervention

- `Amazon WorkDocs` is commonly used to easily collaborate, share content, provide rich feedback, and collaboratively edit documents with other users.



- `Expedited retrievals` allow you to quickly access your data when occasional urgent requests for a subset of archives are required. For all but the largest archives `(250 MB+),` data accessed using `Expedited retrievals` are typically made available within `1–5 minutes`. `Provisioned Capacity` ensures that retrieval capacity for `Expedited retrievals` is available when you need it.
- `Provisioned capacity` ensures that your retrieval capacity for `expedited retrievals` is available when you need it. Each unit of capacity provides that at least three `expedited retrievals` can be performed every five minutes and provides up to `150 MB/s` of retrieval throughput. 

- `Amazon FSx File Gateway` to support the `SMB` file share for the on-premises application

- `AWS Artifact `is your go-to, central resource for compliance-related information that matters to you. It provides on-demand access to AWS’ security and compliance reports and select online agreements

- `AWS Transfer for SFTP` enables you to easily move your file transfer workloads that use the Secure Shell File Transfer Protocol (SFTP) to AWS without needing to modify your applications or manage any SFTP servers.

- `Amazon S3 access points` simplify data access for any AWS service or customer application that stores data in `S3`. `Access points` are named network endpoints that are attached to buckets that you can use to perform S3 object operations, such as `GetObject` and `PutObject`.

- `CloudFront Match Viewer` is an Origin Protocol Policy that configures `CloudFront` to communicate with your origin using HTTP or HTTPS, depending on the protocol of the viewer request

- `An Amazon S3 Glacier (Glacier) vault` can have one resource-based vault access policy and one `Vault Lock policy` attached to it. A `Vault Lock policy` is a vault access policy that you can lock. Using a `Vault Lock policy` can help you enforce regulatory and compliance requirements

- `SageMaker Clarify `is a tool designed for detecting bias and providing explainability for machine learning models


- `Babelfish for Aurora PostgreSQL` is an extension designed for Amazon Aurora PostgreSQL. It allows the database to interpret commands from applications developed for Microsoft SQL Server. By serving as a translation layer, it converts T-SQL (Microsoft’s SQL dialect) into PostgreSQL, enabling SQL Server applications to run directly on Aurora PostgreSQL with minimal or no changes required to the code.

- `Amazon Comprehend Medical` uses advanced machine learning models to accurately and quickly identify medical information such as medical conditions and medication and determine their relationship to each other, for instance, medication and dosage. You access `Comprehend Medical` through a simple API call, no machine learning expertise is required, no complicated rules to write, and no models to train.


-` AWS Health` provides ongoing visibility into your resource performance and the availability of your AWS services and accounts. You can use` AWS Health` events to learn how service and resource changes might affect your applications running on AWS.` AWS Health` provides relevant and timely information to help you manage events in progress.` AWS Health` also helps you be aware of and to prepare for planned activities.


- `AWS Systems Manager Run Command` lets you remotely and securely manage the configuration of your managed instances. A managed instance is any Amazon EC2 instance or on-premises machine in your hybrid environment that has been configured for Systems Manager



- `Karpenter `is a flexible, high-performance Kubernetes cluster autoscaler that launches appropriately sized compute resources, like `Amazon EC2 instances`, in response to changing application load. It integrates with AWS to provision compute resources that precisely match workload requirements.


- `EFA` provide OS bypass ability for Linux systems
    - not supported for windows
    - supports HPC workloads

- you can’t assign an Elastic IP address to an `Application Load Balancer`



- `Inspector`: Vulnerability management. Finds weaknesses in your configurations.
- `Detective`: Threat investigation. Analyzes security events to understand what happened.
- `GuardDuty`: Threat detection. Continuously monitors for malicious activity and unusual behavior

- `AWS PrivateLink` (which is also known as VPC Endpoint) is just a highly available, scalable technology that enables you to privately connect your VPC to the AWS services as if they were in your VP

- `VPC` endpoints for `S3` and `DynamoDB`
    -  `gateway` - used for AWS Services
    - `interface` - used for external services
        -  an elastic network interface with a private IP address from the IP address range of your subnet. Unlike a Gateway endpoint, you still get billed for the time your interface endpoint is running and the GB data it has processed. 

![text](images/exam-prep/multi-az-vs-read.png)
![text](images/exam-prep/ec2-types.png)

## From Official AWS Practice Exam ##

- `Amazon Cognito` provides authentication, authorization, and user management for your web and mobile apps. Users can sign in directly with a user name and password, or through a trusted third party
- You can use `AWS STS` to create and provide trusted users with temporary security credentials that can control access to your AWS resources. However, `AWS STS` does not control access to an application.
- For `SnowBalls Edged Device `base price covers the device and 10 days of usage at the on-premises location. If the company returns the device within a week, the company pays the base price and the price for data transfer out of AWS.
- `AWS Shield` provides protection against DDoS attacks. Shield cannot be a target of routes in a route table
- The only way to retrieve instance metadata is to use the `link-local address`, which is 169.254.169.254.
- `Amazon GuardDuty` is a threat detection service that continuously monitors for malicious activity and anomalous behavior. It does not scan for vulnerabilities.
- You can use `Patch Manager` to apply patches for operating systems and applications
- `Amazon Inspector `removes the operational overhead that is necessary to configure a vulnerability management solution. `Amazon Inspector `works with both EC2 instances and container images in Amazon ECR to identify potential software vulnerabilities and to categorize the severity of the vulnerabilities
- This is a pilot light DR strategy. This solution recreates an existing application hosting environment in an `AWS Region`. This solution turns off most (or all) resources and uses the resources only during tests or when DR failover is necessary. RPO and RTO are usually 10s of minutes.
-  By default, the Application Load Balancer waits 300 seconds before the completion of the deregistration process, which can help in-flight requests to the target become complete. To change the amount of time that the Application Load Balancer waits, update the deregistration delay value.
- Components of a VPN connection:
    - A `customer gateway` is required for the VPN connection to be established. A customer gateway device is set up and configured in the customer's data center.
    - A` virtual private gateway` is attached to a VPC to create a `site-to-site VPN connection to AWS`. You can accept private encrypted network traffic from an on-premises data center into your VPC without the need to traverse the open public internet.