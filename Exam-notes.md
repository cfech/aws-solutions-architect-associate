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

- File Gateway = SMB

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