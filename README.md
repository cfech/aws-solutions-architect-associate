# AWS Certified Solutions Architect Associate SAA-C03 Course Code
*By [Digital Cloud Training](https://digitalcloud.training/) - Course Author Neal Davis*


**Sections**
1. [Global Services Index](#global-services)
2. [Identity and Access Management (IAM)](#iam)
3. [Elastic Compute Cloud (EC2)](#ec2)
4. [Elastic Load Balancing and Auto Scaling](#elastic-load-balancing-and-auto-scaling)
5. [AWS organizations](#aws-organizations)
6. [AWS Virtual Private Cloud (VPC)](#aws-virtual-private-cloud-vpc)
7. [Amazon Simple Storage Service (S3)](#amazon-simple-storage-service-s3)
8. [DNS, Caching and Performance Optimization](#dns-caching-performance-optimization)
9. [Block and File Storage Systems](#block-and-file-storage)
10. [Docker Containers And ECS](#docker-containers-and-ecs)
11. [Serverless Applications](#serverless-applications)
12. [Database and Analytics](#database-and-analytics)
13. [Deployment And Management](#deployment-and-management)
14. [Monitoring, Logging and Auditing](#monitoring-auditing-and-logging)
15. [Security In teh Cloud](#security-in-the-cloud)
16. [Migration and Transfer](#migration-and-transfer)
17. [Web, Mobile, ML, Cost Management](#web-mobile-ml-cost-management)

## Global Services ##
- IAM
- CloudFront
- Billing and Cost Management
- Route53
- AWS Organization
- AWS Artifact
- AWS Health Dashboard
- S3 (from a bucket name perspective)

## IAM ###
- https://digitalcloud.training/aws-iam/
- Users- `IAM` users are individuals or applications that you create in AWS to grant them access to your AWS resources.
    - long term
- Groups- `IAM` groups are collections of `IAM` users that you can use to simplify permissions management by assigning permissions to the group instead of individual users.
- Roles- `IAM` roles are sets of permissions that you can assign to AWS resources (like `EC2` instances) or federated users, allowing them to assume specific permissions without having long-term credentials.
    - short term
    - have `Trust Relationships` which define who/what can assume this role
- Policies- `IAM` policies are documents that define permissions for users, groups, or roles, specifying which actions they can perform on which resources.
    - Policies are the rules.   
    - Users, groups, and roles are who the rules apply to.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

### Best Practices ###
- Do not use the root user
- Create an admin user, then create a regular user to take actions with 
  - this follows principle of least privilege and only use the account with the least amount of privilege required
- Create groups to put users into for easy reassignment of permissions 
- see: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html


### Auth Methods ###
- MFA - 2 of the following
    - something you know - ie: passwords
    - something you have - ie: token/authenticator
        - security key
        - mobile auth app
        - time based on time passwords (TOTP)
    - something you are- ie: biometrics 
- `Access Keys` - programmatic 
    - used for CLI or other API endpoints
    - `Access Key ID` and `Secret Key`

### Permission Boundaries ###  
- Sets maximum permissions an entity can have
- Assigned to users and roles
- Even if you have a policy, the boundary will restrict the max permissions the user can have
- Helps to mitigate privilege escalation
  - ie: something who only has `IAM` flags could create a user and given them admin access
    - can give someone `IAM` Full access but add a permission boundary that says the users this user creates can only have same or lower permissions as the current user

### Evaluation Logic ###
 ![text](./images/iam/permission-logic.png)
1. explicitly deny
2. organization policy
3. resource policy
4. identity policy
5. `IAM` permission boundary
6. Session policy

![text](./images/iam/authorizing-in-aws.png)
![text](./images/iam/types-of-policy.png)

- Requests are implicitly denied by default 
    - Root user has full access
- Explicit deny overrides any allows 


### IAM Policy Structure ###
- Everything in AWS is an api call
- Each service has a set of actions
    - example `"Action": "s3:GetObject"`
- `IAM` Policies are are written in JSON
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "s3-object-lambda:WriteGetObjectResponse"
            ],
            "Resource": "*"
        }
    ]
}
```
- The only effects are `allow` or `deny`
- Action - list specific operations this policy affects
- Resource - which resource these actions can be performed on
- `*` - wildcard - signifies all from this point


## EC2 ##
- https://digitalcloud.training/amazon-ec2/
- Virtualized servers run in AWS data centers
    - use Zen and Nitro virtualization software
    - Virtualization Stack - IAAS
- Chose the OS, Instance Type, resources
- Can be assigned public IP
    - Assigned automatically when in a `public subnet`
    - IP is lost when instance is stopped
    - Associate with a private instance address
- Call instances have private IP address
- Can assign an `elastic ip`, which is a static public IP
    - you are charged while it is not used

### Subnet Types ###
- `AZ` can hold subnets, private and public
- Instances can be launched into `private subnets`
    - should deploy here whenever we can 
    - can then use a `load balancer` in a `public subnet` to route traffic to them
    - Can deploy a `NAT Gateway` in the `public subnet` for downloading things from the internet like dependency updates
    - `Nat Gateways` Only Allow outbound traffic
- `Subnets` have routes assigned to them
    - Public Subnet Table
        - `172.31.0./15 : local`
        - `0.0.0.0 : igw-id` (mapped to the `internet gateway`)
    - Private Subnet table 
        - `172.31.0./15 : local`
        - `0.0.0.0 : nat-gateway-id` (lives in `public subnet` which then send traffic to internet gateway)

### Launching EC2 ###
- chose [instance type](https://aws.amazon.com/ec2/instance-types/)
- chose AMI (includes OS and software)
    - back by a snapshot (point in time backup)
    - create create our own custom `AMI`

### Instance Metadata ###
- can be curled by `curl http://[ip]/latest/meta-data
- includes various data such as AMI-ID, hostname, etc...
- 2 versions of the instance metadata service
    - v1 - older and less secure
    - v2 - default, newer, more secure, requires authorization via session token
- `User Data` - ability to supply a script to run when the VM launches
    - can be configured in the management console or cli
    - in the cli the user data is passed in via file
    - must be base64 encoded
    - limited to 16kb
    - only runs the first time the instance is launched
- In AWS `EC2`, `169.254.169.254` is the default gateway for instances. It's a virtual IP address that allows instances to access the Instance Metadata Service (IMDS).
    - ex: `curl http://169.254.169.254/latest/meta-data/instance-id`
        - will give us the instance id
    - see [user-metadata-md](./aws-saa-code-main/amazon-ec2/user-data-metadata.md) for more

### User Data ###
- can be given to `EC2` as a script to run on startup in order to configure the instance
- always converted to base64
- only runs one time when the instance starts
- max 16kb


### Access Keys vs Roles With EC2 ###
- AWS CLI uses access keys
    - Access Keys are associated with a user account and inherit that users permissions
    - **This can give `EC2` instance permissions but access keys are long term credentials and stored in plain text**
    - Access keys are configured with `aws configure`
- Should instead use `IAM` roles for services
    - temporary credentials where permissions are inherited just for the time that is required to perform the action
    - nothing is stored on the instance 
    - `EC2 Assumes the role`
    - These credentials are like short term access keys that are renegotiated when the role needs to be assumed.

### Status Checks And Monitoring ###
- Status Checks 
    - System Status Check - checking if underlying hardware is running
        - aws is going to have to fix this issue
    - Instance Status Check - checking the software and networking configuration
        - you may be able to fix this issue
- Monitoring
    - the monitoring tab shows various metrics that are sent to the `AWS Cloudwatch` service
    - By default metrics are sent to `CloudWatch` every 5 mins for free
    - `Detailed Monitoring` can be enabled for more frequent (every 1 minute) reporting but this will cost $$    
        - `CloudWatch` detailed monitoring is enabled by default when creating launch configurations through the CLI.

### EC2 Placement Groups ###
- A way to control how AWS deploys `EC2` and what `AZ` they are in
![text](./images/ec2/cluster-pg.png)
- `Cluster` - packs instances closely within the same `AZ`, as opposed to randomly
    - achieves low-latency network performance
    - good for tightly couple node to node communication for HPC applications 
![text](./images/ec2/parition-pg.png)
- `Partition` - spreads instances across logical partitions
    - redundancy, so machines in the same partition do not share the same hardware
    - good for distributed workloads such as a HDFS, Cassandra and KAfka
![text](./images/ec2/spread-pg.png)
- `Spread` - places a small group of instances across distinct underlying hardware
![text](./images/ec2/placement-group-use-cases.png)

### Network Interfaces ###
- 3 Types
    - `ENI` - elastic network interface
    - `ENA` - elastic network adapter
        - must have the ENA module installed and enabled
            - Amazon Linux has this by default
    - `EFA` - elastic fabric adapter
        - Used for HPC
- when launching an `EC2`, the `EC2` sits in the `AZ` and it attaches to an Adapter in the subnet
- can connect multiple Adapters from different subnets to the same `EC2` but the subnets must be in the same `AZ`
- Cannot remap adaptors across `AZ`
![text](./images/ec2/network-interface-types.png)

### Types of IPs ###
- Public - dynamic, changes with stops and starts of instances
- Private - used in `private subnet` only
- `Elastic IP` (EIP) - a public ip that is static
    - can be assigned to network interfaces
    - can be remapped across `AZ`'s
    - you get charged when these are allocated to the account but not in use

###  NAT For Public Addresses ###
- Network Address Translation (NAT)
- Public IP addresses do not exist anywhere on the `EC2`
- The public address is translated to the private and vice versa by the `Internet Gateway`
![text](./images/ec2/nat.png)

### Private Subnets and Bastion Hosts ###
- https://digitalcloud.training/ssh-into-ec2-in-private-subnet/
-  A `bastion host` in AWS is a special-purpose instance that acts as a secure gateway to your private network within a Virtual Private Cloud (VPC). It sits in a public subnet, meaning it has a public IP address and can be accessed from the internet.
- Within a region we have a vpc, `AZ` and `public subnet`
- the `public subnet` has a route table in it which has a route to the local network and route to public internet 
- `private subnet`s cannot be connected to the internet, there is no route option
    - this is the default for new subnets

![text](./images/ec2/bastion-host.png)
- can launch 2 `EC2`s, 1 in `public subnet` and 1 in `private subnet`
    - have an internet gateway and want to communicate from the outside to a computer in the `private subnet`
    - can send a request to the `EC2` in the `public subnet` which can then route the request from internet, across the private ip routes 

### NAT Gateways and Instance Overview ###
- Can be used with bastion host for outbound only connection
    - ie download software or outside service
    - no one from the outside should be able to connect
- We can deploy the `NATGateway` into the `public subnet` so that it can get it a public ip
    - this will allow it to proxy traffic from the private `EC2` to the internet
    - **alway deploy nat gateways or nat instances into a public subnet**
    - the `NatGateway` ID is added to teh `private subnet` route table 
    - Managed service
    - Can scale horizontally
    - Has high availability, can be deployed in multi-az
    - outbound access only, no security groups
    - **needs elastic ip**
    - need to add a route to the route table in private route table

![text](./images/ec2/nat-gatway-for-bastion-host.png)
- `NAT Instance` - an `EC2` that does the same thing as a `NatGateway` but runs from and `EC2` instance
    - not used much anymore
    - uses special ami `amzn-ami-vpc-nat` 
    - **must disable source and destination checks**
    - **managed by you!!**
    - can only scale out
    - not highly available
    - uses security groups
    - could make this dual purpose so it is actually your bastion host if you wanted 

![text](./images/ec2/nat-instance.png)
![text](./images/ec2/nat-instance-vs-gateway.png)

### EC2 Instance LifeCycle ###
![text](./images/ec2/instance-life-cycle.png)
- Stopping
    - Can only stop instances for EBS backed instances
    - `EC2` does not charge for stopped instance but EBS volumes do charge
    - Data in ram is lost
    - Instance is migrated to a different host
    - Private IPV4/6 are retained, public IPV4 are released
- Hibernating 
    - applies to supported AMI
    - contents of RAM saved to EBS volume
    - Must be enabled for hibernation at launch
    - Specific pre-reqs apply
    - When started (after hibernation)
        - Ebs root volume is restored
        - RAM is reloaded
        - Processes resumed
        - Data volumes reattached
        - Retains instance ID
- Rebooting 
    - equivalent to OS reboot
    - DNS named and all IPV4/6 addresses are retained
    - Does not affect billing
- Retiring
    - Instance may be retired if AWS detects irreparable failure to hardware
    - Stopped or terminated when retirement time is reached 
- Terminated
    - deleting and instance
    - cannot recover
    - by default root EBS volumes are deleted
- Recovering 
    - `CloudWatch` can be used to monitor system status and recover if needed
    - Applies if the instance becomes impaired due to underlying hardware / platform issues
    - Recovered instance is identical to original instance

### Nitro Instances and Nitro Enclaves ###
- Nitro is the underlying platform for next generation of `EC2`
- Supports virtualized and bare metal instances
    - Trying to eliminate virtualization performance instances
- The AWS Nitro system does have dense storage instance options
- Specialized hardware 
    - Nitro cards for VPC
    - Nitro cards for EBS
    - Nitro for Instance Storage 
    - Nitro card controller
    - Nitro security chip
    - Nitro hypervisor
    - Nitro Enclave
- Improves performance, security and innovation
- Nitro Enclaves
    - isolated compute environments
    - hardened vms
    - no persistent storage, external access or interactive access
    - cryptographic attestation
    - integrates with aws KMS (Key Management Service)
    - Protect and securely process sensitive data like PII

### EC2 Pricing Options ###
![text](./images/ec2/pricing-options.png)
- Commercial linux distros such as RHEL use hourly pricing 
    - minimum of 1 hour
- Amazon linux and other OS are biller per second with a minimum of 1 minute
- `EBS volumes` are per second with a min of 1 minute
- Reserved instances
    - term is 1 or 3 years
    - standard and convertible RI
        - standard- change `AZ`, instance size and network type with `ModifyReservedInstances` API
        - Convertible - can change all of standard and family, OS, tenancy, payment option with `ExchangeReservedInstances` API
    - can pay all up front, partial up front, no up front
    - can reserve capacity in a specific `AZ` but does not reserver capacity when specified by region
    - tenancy can be default or dedicated 
- On Demand Capacity reservation 
    - reserve in specific `AZ`, for any duration
    - mitigates risk of being unable to get On-Demand Capacity
    - Does not require any term commitments and can be cancelled at any time 
- Savings Plans 
    - Compute Savings Plan-  1 or 3 year, hourly commitment to usage of `Fargate`, `Lamdba` and `EC2`, any region, family, size, tenancy and OS
    - `EC2` Savings Plan-  or 3 year, hourly commitment to usage of `EC2`, within a selected region and instance family; any size tenancy and OS
- Spot Instances 
![text](./images/ec2/spot-instances.png)
- can ask for a spot block to get 1-6 hours of uninterrupted compute
- pricing is 30% to 45% discount 

### EC2 Pricing Use Cases ###
![text](./images/ec2/prcing-use-cases.png)

## Elastic Load Balancing and Auto Scaling ##
- https://digitalcloud.training/auto-scaling-and-elastic-load-balancing/

### Scaling Up vs Scaling Out ###
- Stateful applications - the application needs to remember user information
    - eg: shopping site
- Stateless application - no state about the user is remember by the app
    - eg: weather app
- Generally want to keep the web server stateless by putting user data in cookies, caches and dbs

- Scaling up - add more resources to the same server
    - ie: changing the instance type of the `EC2`
    - SQL DB's are generally scaled up
- Scaling out - add more servers of the same size
    - ie: running more of the same instance
    - static web servers are generally scaled out 


### Amazon EC2 AutoScaling ###
![text](./images/lb/auto-scaling.png)
- Automatically launch or terminate instances based on capacity and health
- Works with `EC2`, `ECS`, `EKS`
- Integrates with:
    - `CloudWatch` for monitoring and scaling
    - `Load Balancer` for distributing connections
    - `VPCs` to be deployed in different `AZ`
- Autoscaling - scales horizontally providing elasticity and scalability
    - Responds to `EC2` Status Checks
    - Can scale on demand or schedule
    - Scaling policies define how to respond to changes in demand


![text](./images/lb/asg.png)
- Need to setup and Auto Scaling Group
- Launch Template - specifies the instance configuration
    - additionally health checks, vpc, subnets, purchase options etc...
- Health checks:
    - `EC2` - comes with health checks checking if the machine is ok
    - `ELB` - load balancer is checking if the instances are alive
    - `Health Check Grace Period` - how long to wait until the health is checked
        - Auto scaling does not act until this has been satisfied
    - You should use both `EC2 status checks` and `ELB health checks`. If you select `ELB health checks` both are used automatically. This configuration ensures that the `ELB` does not forward traffic to instances determined by `EC2` status checks to be unhealthy.
- Auto Scaling types
    - Manual - make changes manually
    - Dynamic - based on demand 
    - Predictive - uses ML
    - Scheduled - based on schedule 

- Steps
    1. Create a `launch template`
        - defines what to run when ASG scales
    2. Create an `ASG`
        - will spread the instances across the `AZ`s by default
        - can create dynamic, scheduled or predictive scaling policies
        - of there are less instances than the min alloted it will launch new instances 
            - this happens based on `EC2 health checks`
    3. Create the `target group`
        - do not statically assign targets
    4. Create the `LB`
        - LB and auto scaler must have the same subnets to work together correctly
    5. Once complete go back to `ASG` and assign the `load balancer` to be able to automatically assign targets

### High Availability and Fault Tolerance ###
- High availability - minimal service interruption, no spof, uptime measured in 9's
    - Services to use aer ELB, Auto Scaling and Route 53
- Fault Tolerance - no service interruption, specialized hardware, no downtime
    - Services to use - fault tolerant NIC, disk mirroring (RAID), db replication and redundant power 

![text](./images/lb/ha-1.png)
![text](./images/lb/ha-2.png)

- When a server fails aws Auto Scaling will replace the server 
- If an entire `AZ` fails we still have 2 web servers alive in a different `AZ`
- The Load Balancer will make sure to route traffic appropriately
- Durable - how often data is lost
- Available - how often we can access the data 

### Elastic Load Balancing ###
- Provides high availability
- Can target - `EC2`, ECS, IP Addresses, lambdas etc...
![text](./images/lb/ha-2.png)
- LBs listen to a target group
    - will keep track of instances with health checks
    - reconnect users if instances go down
    - can tell auto scaling that an instance needs to be restarted 
- Will try and spread the load across available instances
- Auto scaler notifies the ELB when a new instance is launched
- Types:
    - `Application LB` - Http/s, layer 7, path based routing 
        - used for web apps, microservices, docker containers, lambdas
        targets can be instance, ip lambda
    - `Network LB` - TCP/UDP, layer 4, high performance and low latency, tls offloading at scale, static ips in each EZ
        - used for TCP and UDP, low latency, static IP and VPC endpoints
        - targets can be instance, ip and `ALB`
    - `Gateway LB` - Layer 3, virtual appliances like firewalls, IPS, IDS
        - Deploy scale and managed 3rd party network alliances, provides centralized monitoring
        - Uses the GENEVE Protocol on 6081

### ALB and NLB Deployments ###
- Instances are registered in target groups
![text](./images/lb/elb1.png)
![text](./images/lb/elb2.png)


- `ALB` offers advance request routing such as path routing
    - can look at information in http header and send to different target groups based on this info
    - can also do host based routing 
- `NLB` can forward traffic based on ips
    - can assign static ip in each subnet 
    - routed based on ip protocol data such as port number 

![text](./images/lb/elb3.png)
- `ALB` uses the private ip of the LOAD Balancer of the source 
- `NLB` uses the source IP of the client making the request if the instance is specified by ID
    - the NLB uses its private IP if the instance is specified by IP address and using HTTP/S
- Can enabled the `x-forwarded-for` header to alway keep track of the original client ip


### EC2 Scaling Policies ###
![text](./images/lb/target-tracking.png)
- Target Tracking - scales based on a target metric value such as average CPU usage across the `AZ`, aims to keep metric close to the target
    - recommended by AWS
    - recommended 1 minute frequency
- Simple Scaling - scales based on an alarm, when cpu > 60% for example it will launch a new instances
- Step Scaling - similar to simple, the amount of instances launched is relative to th size of the breach 
- Scheduled Scaling - set a specific time when to launch a specific number of instances 

| Feature | Target Tracking Scaling | Simple Scaling |
|---|---|---|
| **Scaling Trigger** | Target value for a metric | CloudWatch alarms |
| **Scaling Adjustment** | Automatic, based on algorithms | Predefined, based on alarm configuration |
| **Configuration** | Define target value and metric | Define alarms and scaling adjustments |
| **Proactive Scaling** | Yes | No |
| **Granular Control** | Less | More |


### Cross Zone Load Balancing ###
![text](./images/lb/cross-zone.png)
- When enable each load balancer distributes traffic across register targets in all enabled `AZ`s
    - leads to more even distribution of traffic, especially if the compute is not evenly distributed
- WHne no enabled it only distributes traffic across targets in its own `AZ`
- `ALB` has this enabled
- `NLB` and `Gateway LB` are disabled by default


### Session State and Session Stickiness ###
![text](./images/lb/stick-session.png)
- Session state can be stored in an external db or cache
    - `elasticache` and `dynamoDb` are used because of low latency and key value store
- Sticky sessions mean the LB should always direct the user to the same instance for the life fo the cookie
    - if the instance fails the user may have to restart
    - can utilize with externalizing the state

### Secure Listeners for ELB ###
![text](./images/lb/secure-lbs.png)
- certificates to LB's for SSL and TLS
- `ALB` - supports SSL/TLS Cert
    - If w want encryption all the way to the application instance we will need a certificate at both the LB and the `EC2`
- `Network LB` support ssl/tls certs on the `EC2` for complete encrypted connection 
- Can use `AWS Certificate Manager` to help request certificates for `Route 53 Hosted Zones`


## Aws Organizations ##
![text](./images/org/organization-overview.png)

- https://digitalcloud.training/aws-organizations/
- create one organization for many accounts
- allow for `service control policies (scp)`
- can consolidate multiple accounts into an organizations that allows for centrally management
- Two Feature Sets:
    - `Consolidate Bill` - only 1 main account gets the bill, 1 card
        - Aggregates usage of services across all accounts which can come with some discounts
        - Includes paying account 
            - independent and cannot access resources from other accounts
            - Credit card is linked to
        - `Linked Accounts`
            - all linked accounts are independent and can use services 
    - `All Features` - gives some additional capabilities including `SCP`
- `Management accounts`, create organizations and add accounts (aka `Master/Management Accounts`)
    - this does not have to be the overall root aws account
    - can group users into organizations units
    - can create account programmatically
    - can use `AWS SSO`
    - Allow for use of `service control policies` to limit permissions per account 
    - The `root` is the top-most container in an `AWS organization's` hierarchy, while the management account is the account that creates the `organization`
- Policies are applied to root accounts or `OUs`

- There are many services and policies that are disabled by default when creating an `organization`
    - must explicitly enable them



### Organization Accounts 

![text](./images/org/organization-overview.png)

| Feature | Management Account | Member Account |
|---|---|---|
| **Authority** | Highest level of control | Operates within defined boundaries |
| **Creation** | Creates the organization | Created by the management account or invited to join |
| **Policies** | Defines and applies policies | Inherits and implements policies |
| **Billing** | Manages consolidated billing | Contributes to consolidated billing |
| **Access Control** | Sets organization-wide access controls | Manages individual account access |


### Service Control Policies
![text](./images/org/scp.png)
- SCP control maximum available permissions, they are explicit deny
    - **the do not grant permissions , just limit them**
- SCP can be applied at the OU level 
    - in the example the dev ou can only launch t2.micro
    - the policies flow down stream through the organizational hierarchy
    - **these constraints apply to everyone including admins**
- OUs can be created and have users added to them
- Example SCP
```
{    
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAccessToASpecificRole",
      "Effect": "Deny",
      "Action": [
        "iam:AttachRolePolicy",
        "iam:DeleteRole",
        "iam:DeleteRolePermissionsBoundary",
        "iam:DeleteRolePolicy",
        "iam:DetachRolePolicy",
        "iam:PutRolePermissionsBoundary",
        "iam:PutRolePolicy",
        "iam:UpdateAssumeRolePolicy",
        "iam:UpdateRole",
        "iam:UpdateRoleDescription"
      ],
      "Resource": [
        "arn:aws:iam::*:{role/name-of-role-to-deny}"
      ]
    }
  ]
}
```
- `Tag Policy` - Tag policies allow you to standardize the tags attached to the AWS resources in an organization's accounts.
    - You can use tag policies to maintain consistent tags, including the preferred case treatment of tag keys and tag values.




### AWS Control Tower
![text](./images/org/control-tower.png)
- **extends capabilities of organizations** 
- released after organizations
- sits over the top to add some guardrails
- creates a `landing zone`
    - a well architected multi-account baseline
    - has a whole bunch of preventive guardrails that disallow api actions using SCP
- Integrates with SSO

- Shared accounts with control tower
    - Management Account
    - Log archive account 
        - stores copy of all `aws cloudtrail` and `aws config` log files
    - Audit account
        - aggregates and stores logs collected from other accounts in the landing zone
        - secure account with restricted access
- Preventative Guardrails
    - designed to prevent policy violations before they occur
    - ie: `SCP`
    - Example - disallow deletion og `Cloudtrail Logs` and `S3 Logging Buckets`
- Detective Guardrails
    - monitor and report policy violations for non-compliant actives that have already occurred
    -  help identify mis conduct 
    - Use `AWS Config` rules and Lambda Functions 
    - Continuos Evaluation
    - Example - detect publicly accessible `S3 Bucket`

![text](./images/org/organization-vs-control-tower.png)



## AWS Virtual Private Cloud (VPC) ##
- Can create 5 VPC's per `Region`
- Each `Region` has a default `VPC`
- Avoid overlapping CIDR Blocks
- https://digitalcloud.training/amazon-vpc/
- https://digitalcloud.training/aws-direct-connect/

### AWS Global Infrastructure ###
![text](./images/vpc/regions.png)
- `Regions` hold `Availability Zones (AZ)`
    - distant from each other
- `AZ` are 1 or more physical data centers 
    - have redundant power sources and networking
    - fairly close to each other
- Common to have resources in the same `Region` and Multiple `AZ` as well as in different `Regions` for high availability
- `Regions` are connected by `AWS Local Network`
- `Subnets` are crated within `AZ` 
    - Public Subnets:
        - Must have a route to and `Internet Gateway`
        - Must have `Auto-assign public IPv4 address` set ot `yes` 
- `Direct Connect` 
- `Outposts` - piece of hardware that comes into your data center
    - supports subset of AWS
    - Has connectivity back to AWs region
- `AWS Local Zone` - like AZ in metro politian areas
    - can provide closer connectivity areas
- `Wavelength zones` - for 5G, goal is to lower the delay and latency to your data center

![text](./images/vpc/az.png)

- `Amazon Cloudfront` - aws CDN
    - Have `Regional Edge Cache` 
    - Allows for users around the world to access resources at lower latency
- AWS Allows us to simply deploy services globally from the management console. cli or api

### IP Addressing ###

![text](./images/vpc/ip1.png)
![text](./images/vpc/ip2.png)


### Amazon VPC Overview ###
![text](./images/vpc/vpc-diagram.png)
- `Regions` have `VPC`
- `VPC` cannot span `Regions`
- Logically isolated portion of the AWS cloud withing a `Region`
- `AZ's` can be used withing the `VPC`
- `Subnets` are created and aligned to `AZ`
    - cannot assign the same `Subnet` to multiple `AZ`
    - Can have multiple `Subnet` in the same `AZ`
- `VPC Router` - handles routing within and out of the `VPC`
    - You don't handle this, but can configure the `VPC` Rout table
- `Internet Gateway` attaches to `VPC` and used for egress/ingress to the internet
    - only 1 per `VPC`
    - There is an `Egress Only Internet Gateway` that uses IPV6 and only allows egress
- Can create multiple (Up to 5 by default) `VPCs` in a `Region`
    - each `VPC` has its own CIDR block
    - can create the network ID's for the `Subnets`, but they have to be within the CIDR block

![text](./images/vpc/vpc-components.png)
- `Peering connection` - connects multiple `VPCs`
- `VPC Endpoint` - private IP can connect to AWs
- `Virtual Private Gateway` and Customer Gateway - deal with creating VPN connections
- `AWS Direct Connect` - allow high speed, private network connection to AWS
    - Avoids public internet


![text](./images/vpc/vpc-core-knowledge.png)
- **VPC is like having your own Data Center in AWS**
    - needs a CIDR blocks
    - Spans all `AZ` in a `Region`
    - Provides complete control over virtual networking 
    - Can create 5 per `Region`
    - Full Access over AWS resources 
    - Default VPC os created in each `Region` with a public `Subnet` by default 

### Defining VPC CIDR Blocks ###
![text](./images/vpc/cidr-rules.png)
- Size should be between /16 and /28
- Cannot overlap an existing `VPC`
- cannot increase eor decrease and existing size
- **first for and lastIP address are not available for use**
    - this is done at the subnet level
- recommend you use private ip ranges
- `VPC Subnets` have a longer subnet mask than the `CIDR` block they come from

| IP Address | Purpose | Example (CIDR block 10.0.0.0/24) |
|---|---|---|
| Network address | Identifies the subnet itself | 10.0.0.0 |
| VPC router address |  Used by the VPC router for routing traffic | 10.0.0.1 |
| DNS server address | Used for DNS resolution within the VPC | 10.0.0.2 |
| Future use address | Reserved for future use by AWS | 10.0.0.3 |
| Network broadcast address | Used for broadcast communication within the subnet | 10.0.0.255 |


![text](./images/vpc/cidr-considerations.png)
- Bigger blocks give more flexibility
- Smaller `subnets` are OK for most use cases
- Consider deploying applications tiers per `subnet`
- `VPC` peering requires non-overlapping cider blocks
- **Avoid overlapping CIDR blocks**
- Example subnet calculator: https://www.site24x7.com/tools/ipv4-subnetcalculator.html

### Custom VPC ###
![text](./images/vpc/custom-vpc.png)
- important thing to know here is the difference in the rout tables
- private `subnet` has no connection to the `igw`


### Security Groups and Network ACL ###
![text](./images/vpc/sg-vs-nacl.png)


- Stateless firewall - must have explicit rules to allow data flow in each direction
- Stateful firewall - allows return traffic automatically if incoming traffic was allowed 

- `Security Group` 
    - instance level 
    - can be applied to instances in any `subnet`
    - stateful firewall - will allow outbound traffic if inbound is allowed
        - this is because of the `outbound rules`
    - only support allow rules (ie: whitelist)
    - `Security Group Chaining` - set inbound or outbound traffic only from another security group
- `Network Access Control List`
    - apply at the subnet level
    - only apply to traffic entering/exiting the subnet
    - stateless firewall
    - process rules in order
        - rules are numbered, and are evaluated in this order
    - support allow and deny rules
    - apply to all instances in the subnet it is associated with


### Amazon VPC Peering ###
![text](./images/vpc/peering.png)
- Allow routing between `VPCs` via AWS network
- Enables routing using private IPv4/6 addresses
- **CIDR blocks must not overlap**
- Does not support transitive peering, must mesh the network if we want that 
    - see [AWS Transit Gateway](#aws-transit-gateway)
- `VPC` can be in different `accounts` and `regions`
- Must set up `Security Group` protocols and `Route Tables` for this to work


### Creating Peering Connection ###
- create all resources (VPC, Subnet etc,..)
- Configure `security groups` to allow traffic from the other `VPC` CIDR Block
- `VPC` management console select `VPC peering connection` and fill out the details
- In the other `Region/VPC` must accept the peering request
- The must create the route tables in order to have traffic routed to each other
![text](./images/vpc/example-peering-route-table.png)


### VPC Endpoints ###
![text](./images/vpc/internet-and-gateway-endpoints.png)
- `VPC Interface Endpoint` - creates `ENI` in `subnet`
    - allows EC2 to connect to other AWS services but not the internet
    - uses private IP

- `VPC Gateway Endpoint` - uses the **route table** to create a path to the resource
    - `S3` and `DynamoDB` only
    - can allow us to connect from `private subnet` to `s3` bucket without having a public ip
    - can secure this with `IAM` `policies`

![text](./images/vpc/internet-provider-model.png)
- Can create a model where the is a `Service VPC` and `Consumer VPC` for separation of concerns 

- see [these directions](./aws-saa-code-main/amazon-vpc/create-s3-gateway-endpoint.md)

### AWS Client VPN ###
![text](./images/vpc/client-vpn.png)
- Way to connect client computer to a AWS data center `VPC` from your computer
- Encrypted End to End
- Create a VPN Endpoint that associates with `subnets`
- Client computer needs a VPN client
    - Establishes connections via SSL/TLS
    - Performs source network address translation between VPC client and `VPC`

### AWS Site to Site VPN ###
![text](./images/vpc/site-to-site-vpn.png)
- Managed IPSec VPN
- Connect a customer data center/office location into AWS data center
- Allows for use of private IP's 
- AWs creates a `Virtual Private Gateway (VGW)`, customer deploys `Customer Gateway` 
    - allow for an encrypted connection supporting static routes and BGP 
- `Route table` has a route for the CIDR of the external location and send that traffic to the `VGW`
    - Can be used as a backup for `direct connect`



### AWS VPN CloudHub ###
![text](./images/vpc/cloud-hub.png)
- This is not a service but an architecture pattern when using `Site to Site VPN`
- Have a `VPC` and `VGW` but multiple customer locations
    - Can deploy a `Customer Gateway` into each customer location
    - They will each get their own `ASN` (Autonomous System Number )
        - Corresponds with routes advertised into the networks
- Network traffic is bi-directional and could also go between customer locations 
    - using IPSec VPN
- "Connect multiple customer gateways to a VGW to allow for networking of customer locations"


### AWS Direction Connect (DX) ###
![text](./images/vpc/dx1.png)

- Private connection from your data center to `AWS VPC`
- There are locations call `AWS Direct Connect Locations`
    - allow for hard wiring into an AWS DX port 
    - Need a physical connection from the `AWS Direct Connect Location` to your corporate equipment
- Provides private and consistent experience
    - Can be expensive and is only worth it if transferring large amount of data on a regular basis 
- To connect to other private AWS services you need to create a `private VIF (Virtual Interface)` 
- To connect to public services like `S3` you need to create a `Public VIF` in any region
    - does not allow internet connection

![text](./images/vpc/dx2.png)
- to connect to multiple `VPCs` you will need multiple `private VIF`
- **DX connections are not encrypted**
    - Can use an `IPSec Site to Site VPN` over the `DX connection `

- For Dedicated Connections, 1 Gbps, 10 Gbps, 100 Gbps, and 400 Gbps ports are available. 
- For Hosted Connections, connection speeds of 50 Mbps, 100 Mbps, 200 Mbps, 300 Mbps, 400 Mbps, 500 Mbps, 1 Gbps, 2 Gbps, 5 Gbps, 10 Gbps, and 25 Gbps may be ordered from approved AWS Direct Connect Partners.

- regional service, makes sense if there a regional customer locations

### AWS Direct Connect Gateway ###
![text](./images/vpc/dx-multi-region.png)
- Allows the `Private VIF` to connect to multiple `AWS regions `
    - traffic flows over `AWS Backbone`
    - Cannot route regional traffic from 1 region to another
    - Can connect a single office to multiple regions



### AWS Transit Gateway ###
![text](./images/vpc/transit-gateway.png)
- "Cloud Router"
- Helps to solve the `VPC` mesh requirement
- Becomes the network transit hub for all `VPCs` and `Customer Gateways`
    - One subnet is specified from each `VPC`, allow routing in the whole `VPC`
- Can also configure this with `Direct Connect`
    - Requires `Transit VIF`

### Using IPv6 in a VPC ###
![text](./images/vpc/ipv6-1.png)
    - IPv4 - 32 bits - decimal - 4.3 billion addresses
        - NAT used extensively
        - Exhausting the Addresses
    - IPv6 - 128 bits - hexadecimal - over 340 duodecillion (12) addresses
        - 100 for every atom on earth

![text](./images/vpc/ipv6-2.png)
- need both and IPv4 and IPv6 blocks in `VPC`
- need to have a unique hexadecimal pair in the subnet
- have both a local and `igw` route in the `Route Table`
- all IPV6 are publicly routable
- `Egress Only Internet Gateway` - allows IPv6 traffic outbound but not inbound

### VPC Flow Logs ###
![text](./images/vpc/flow-logs.png)

- Capture information about IP traffic to and from network interfaces in `VPC`
    - Have to create a role to allow the VPC to write these logs
    - Then create a flow under `VPC -> Flow Logs`
- Stores in `Cloudwatch Logs` or `S3`
    - would be under Cloudwatch Log Groups

- Can be created at the `VPC`, `Subnet` or `Network Interface` level



## Amazon Simple Storage Service (S3)##
- One of the first AWS services
- **Global service**
- https://digitalcloud.training/amazon-s3-and-glacier/

### Amazon Simple Storage Service
![text](./images/s3/s3.png)

- A `bucket` is a container for objects
- `object` can be any type of files 
- can stores millions of `objects`
    - cheap way to store large quantities of files in the cloud
- Access the `object` with the url
    - the key 0s the name of the `object` or file
- Uses HTTP
- Can use an SDK for interacting with `bucket` in code
- **`Bucket` name must be unique across all of aws**
- S3 sits outside the `VPC` in the public space of `aws`
    - in general if you were to connect to `S3` you would have to reach out to it over the internet
    - in order to connect to `S3` from a `private subnet` you can create an `S3 Gateway Endpoint`

- File Storage creates a hierarchy, function like local storage, maintain connection and are mounted to operating system
    - ie `EFS`
- `S3` is no hierarchical
    - can use `prefixes` to mimic hierarchy
    - Network connection is ended after each request

### Storage Classes ###
![text](./images/s3/storage-classes.png)
- `S3` offers 11 9's of durability
    - if you store 10 million objects then you expect to lose one every 10,000 years
- All `S3` availability is >= 99.5%
    - varies from 99.5% -> 99.99% 
- All designed for durability
- All have different costs 
- `One Zone IA` - only stored in 1 AZ
    - All others have 3+ 
- `S3 Standard` - default
- `Intelligent tiering` - tries to determine the tier for you 
- `Glacier` used for archival purposes
    - `Instant retrieval` - ms latency, more $$ minimum of 90 days
    - `Flexible Retrieval` - mid tier, faster than deep archive (minutes)
    - `Deep Archive` - long term storage for compliance, unlikely to need to access it  (hours)


### IAM Policies, Bucket Policies and ACLs ###
- https://aws.amazon.com/blogs/security/iam-policies-and-bucket-policies-and-acls-oh-my-controlling-access-to-s3-resources/
![text](./images/s3/iam-policies.png)
- `IAM Policies` - identity based policies
    - Specify what actions are allowed on what resources
    - attached to uses groups or roles
    - written in JSON
    - principal is not required
    - Used when:
        - need to control access to AWS services other than `S3`
        - Have numerous `S3` buckets each with different permission requirements
        - Prefer to keep access control policies in the IAM Environment 

- `Bucket Policies` - resourced based 
    - can only be attached to `S3 Buckets`
    - JSON/ `AWS Policy Language`
    - Includes `Principal`
    - Use when:
        - simple way to grant cross account access
        - IAM policies are reaching size limit
        - keep access control policies in S3 environment 

![text](./images/s3/acls.png)
- `Access Control List (ACL)` - legacy control mechanism
    - not recommended by AWS 
    - can be attached to a `bucket` or an `object`
    - limited options for grantees and permissions


### Versioning, Replication and LifeCycle Rules ###
![text](./images/s3/versioning.png)
- Versioning - keep multiple versions of the object in the bucket
    - can revert if you need to (accidental deletion/override)
- Replication 
    - `Cross Region Replication` (CRR) - keep data in 2 different `Regions`
    - `Same Region Replication` (SRR)
    - Versioning must be enabled


![text](./images/s3/lifecycle.png)
- Two Types of actions
    - Transition action - when an object transitions to another storage class
    - Expiration actions - when an object is deleted

- Cannot transition up the stack of s3 offerings
    - Additionally cannot transition from Intelligent Tiering to `One Zone IA`

- Steps
    1. Create 2 buckets
    2. Create and IAM role


### MFA With S3 ###
![text](./images/s3/mfa.png)
- When enabled MFA is required for:
    - Changing the versioning state of a bucket
    - Permanently deleting an object version
- the `x-amz-mfa` request heder must be included in the a requests
- second factor is token from a auth device or program
- verisign can be enabled by:
    - Bucket Owners
    - AWS account that created the bucket
    - Authorized `IAM` users
- MFA delete can only be enabled by the bucket owner

- MFA Protected API access
    - can be use to protect any resource with MFA from API access
    - `aws:MultiFactorAuthAge` key in the bucket policy

### Amazon S3 Encryption ###
![text](./images/s3/encryption-1.png)

- Server Side Encryption with `S3 Managed Keys SSE-S3`
    - `S3` managed keys
    - encryption at rest with AWS encryption
    - secured in transit by HTTPS
- Server Side encryption with `AWS KMS Managed Keys (SSE-kMS)`
    - `KMS` manage keys
    -  AWS or customer managed keys

- Server Side encryption with  `Client Provided Keys (SSE-C)`
    - client manages the keys and not stored in AWS
- Client Side Encryption
    - client managed keys
    - data is encrypted before sending to `S3`


- All `S3` buckets have encryption configured by default
- All uploads encrypted
- No addition cost or performance impact
- Automatically encrypted with `SSE-S3`
- Can use `Batch Operations` to encrypted existing unencrypted files
- Can also encrypt existing objects using the `CopyObject` APi operation or `copy-object` CLI command


![text](./images/s3/encryption-policy.png)
- can use bucket policies to enforce encryption type we want 


### S3 Event Notifications 
![text](./images/s3/sns.png)
- requires an `SNS topic`
    - this `topic` should be allowed to receive events from `S3`

- example Access Policy:
```
{
    "Version": "2012-10-17",
    "Id": "example-ID",
    "Statement": [
        {
            "Sid": "Example SNS topic policy",
            "Effect": "Allow",
            "Principal": {
                "Service": "s3.amazonaws.com"
            },
            "Action": [
                "SNS:Publish"
            ],
            "Resource": "arn:aws:sns:us-east-1:156041403698:email",
            "Condition": {
                "ArnLike": {
                    "aws:SourceArn": "arn:aws:s3:::sns-lab-1231231"
                },
                "StringEquals": {
                    "aws:SourceAccount": "156041403698"
                }
            }
        }
    ]
}  
```
- then need to create an event on the `S3` bucket


### S3 Presigned URLS
- a way to grant access to resources for a period of time without change the access permission of the bucket
- `aws s3 presign s3://sns-lab1231231/presigned-index.html`


### Multipart Upload and Transfer Accelerator ###
![text](./images/s3/multi-part-upload.png)

- `Multipart Upload`
    - perfumed using multipart api
    - recommended for objects 100MB +
    - required for objects > 5GB
    - can be used ofr 5MB - 5TB
    - Key benefits:
        - optimize throughput
        - allow for failure recovery

- `Transfer Acceleration`
    - uses `CloudFront` edge locations to improve performance 
    - upload to an edge location and then traverse AWS Global Backbone to the correct `Region` where the `Bucket` lives
    - Only charged if there is an actual realized benefit from the accelerator



### S3 Select and Glacier Select ###
![text](./images/s3/select.png)
- Can use `S3 Select` to retrieve individual files within a zip in a bucket
    - uses SQL expression
- Glacier Select - for S3 Glacier
- "Use SQL to access objects and objects within objects in S3"


### Server Access Logging ###
![text](./images/s3/server-access-logging.png)

- Allows for logging around events that happen in `S3`
    - Have to enable logging 
    - Want the target to be different then the source
        - If not there would be a loop
- Provides detailed record for requests
- Disabled by default
- Only pay for what storage is used
- Must grant write permissions to the Amazon S3 Log Delivery group 

### S3 Static Website ###
1. Create a bucket
    - must enable it to public access
2. In properties turn on `Static WebSite Hosting`
    - this will generate an http url
    - if we wanted https would have to use other services or put a `cloudfront` endpoint in front on this one
3. Upload index.html and any other files
4. Update the permissions to allow anyone to get the objects in the bucket
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Allow access",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:getObject",
            "Resource": "arn:aws:s3:::website-11111/*"
        }
    ]
}
```


### Cross Origin Resource Sharing (CORS)
![text](./images/s3/cors.png)
- May be required to be enabled in `S3`
    - May need to get some assets to the other bucket
- ie: `example1.com` (origin) sends a request to `resource1.com`
    - running in javascript
    - when a connection request issued a preflight request will be made to check if we are allowed to make a request to the other resource
    - have to explicitly allow this on the other resource 
- Enable through a cors Rule on the bucket we are trying to connect to


### Cross Account Access 
![text](./images/s3/cross-account-access.png)
- Will allow cross account access to assume a role to access an `S3` bucket

- see [example of role](./aws-saa-code-main/aws-iam/cross-account-role.md)

### S3 Object Lambda
![text](./images/s3/s3-lambda.png)

- Uses `Lambda` to process the output of S3 Get requests
- Can use your own functions or pre-built functions
- Need `S3` and `S3 Access points` as well as a `Lambda` function and `S3 Object Lambda Access Point`
    - When a client sends  get request it triggers `lambda` to process the object and return it to the application


## DNS, Caching, Performance Optimization ##
- https://digitalcloud.training/amazon-route-53/
- https://digitalcloud.training/amazon-cloudfront/
- https://digitalcloud.training/amazon-cloudfront/

![text](./images/route53/route53.png)
![text](./images/route53/routing-policies.png)


### DNS and Amazon Route 53 ###
![text](./images/route53/dns-records.png)
- DNS - translates FQDN to IP Address
    -  there is a `.` at the root of the DNS hierarchy

- There are multiple different types of DNS records
    - A  - Ip <-> Domain Name
    - CNAME - Domain Name to another Domain Name
        - `Route53` Charges for queries 
        - Cannot create a CNAM record at the top (zone appex) node of a DNS Namespace
            - only can create sub domains
    - MX - returns mail servers
    - TXT - associates text with the domain name
    - SRV - server locator records
    - NS - name server for the particular domain
    - SOA - `Start of Authority` - stores important information about the domain

- Alias - A DNS record that maps a domain name to a hostname, such as an AWS resource. 
    - `Route53` specific
    - can only point to:
        - `CloudFront Distribution`
        - `Elastic Beanstalk`
        - `ELB`
        - `S3 Bucket with static website`
        - `Another record in the same hosted zone`

- `Route53` - is a dns service
    - can register a public domain name 
    - will create a `Hosted Zone`
        - 2 types of `Hosted Zones`
            - public - determines hos traffic is routed on the internet
            - private - determines how traffic is routed within a `VPC`
                - must set `enableDNSHostName` and `enableDNSSupport` to true
        - endpoints can be domain IP's or Domain Names
    - Can perform health checks 
        -  can chose routing policies based on these health checks
    - Logic for traffic flow to different services
    - Can transfer existing domains into `Route53` if the top level domain is supported
    - Can transfer a domini from `Route53` but must contact AWS
    - Can have a domain in 1 AWS account and `hosted zone` in another account
    - Support Health Checks 

![text](./images/route53/r-policies-2.png)

- `Hosted Zone` - represents a set of records belonging to a domain

- Multiple routing policies
    - Simple - simple DNS response providing IP Address
    - Failover  - if primary is down, route to secondary
    - Geolocation - routes to closes region
    - Geoproximity - closest region in a geographic area
    - Latency - region with lowest latency
    - Multivalue answer - return server IPs and function as a load balance3r
    - Weighted - use relative weights to assigned resources
    - IP Based - route based on the originating IP

| Feature | Geolocation | Geoproximity |
|---|---|---|
| **Routing basis** | User location | User location + resource location |
| **Bias option** | No | Yes (to influence traffic distribution) |
| **Primary goal** | Localized content, regional compliance | Latency optimization, flexible load balancing |



### Route 53 Routing Policies ###

![text](./images/route53/simple.png)
- Simple records in the hosted zone
    - A records with 1+ IP addresses for each domain
    - TTL - how long is the record cached in client DNS cache before refreshing
1. client issues dns query which makes it was to `Route53`
2. `Route53` responds with the IP
3. Client connects to the service with the given IP

![text](./images/route53/weighted.png)
- Create multiple records with the same name but different IPs
- The weights will direct the proportional amount of traffic to each IP
- Weights are defined as a integer between 0 and 255 
- `Route53` supports optional health checks for this strategy and will not send back IP's whose health check failed



![text](./images/route53/latency.png)
- Multiple records with same Domain name
    - Can be `EC2s` or Load balancers etc...
- Health checks occur (optional)
- Depending on where the clients are they get routed to different regions based on latency


![text](./images/route53/failover.png)
- Multiple records with same Domain name, one primary and one secondary
- When query comes in the response will point to the primary if the primary is healthy
- Will be forwarded to secondary if the primary health check fails


![text](./images/route53/geo-location.png)
- Multiple records with same Domain name
- Health checks defined
- In this strategy there is also a region defined, calls will go to the closest region?


![text](./images/route53/multi-value.png)
- Can have multiple records with the same name and multiple IPs
- `Route53` will route connects to these records in a Load balanced fashion
- Will direct to health endpoints only



![text](./images/route53/geoproximity.png)
- Uses `Traffic Flow`, must create a policy 
- Can specify soe coordinates and then point to specific instances based on the coordinates


![text](./images/route53/ip-based.png)
- Create CIDR collection with the CIDR Blocks of the clients
- Can create routing rules which route based on the CIDR collection

### Route53 Resolver ###
![text](./images/route53/resolver.png)

- Works with `Route53` DNS servers and on premises DNS Server
- `Outbound Endpoint`  - connects to DNS server on premises
- `Inbound endpoint` - will allow client in data center to resolve and address in `Route53`

### Amazon CloudFront Origins and Distributions ###
![text](./images/route53/cloudfront-origins.png)

- AWS CDN - service files such as images and videos
- `Origin` - where the content is coming from ie: `S3` or `EC2`
- `Edge Locations` - locations all over the world
    - The content is sent from the origin to the edge locations
    - This is sent over the AWS global network
- Users access the edge locations
    - the goal is to reduce the latency
    - distance is one of the main factors in latency
- Users are directed to nearest edge location
- When you create a `Cloudfront Distribution` you get a domain name
    - can use your own domain name
    - When you create the distribution you set the origin such as `S3` or a custom origin 
    - access via http or https
    - can use live streaming and fill out web forms 
    - Distribution Patterns:
        - Path - someone looking for a specific path go to one origin ver another
        - Viewer - redirect to https?
        - Cache - how long thins stay in the cache
        - Origin Request Policy - configure security and other factors of the request to the origin
    

### Cloudfront Caching and Behavior ###
![text](./images/route53/cfc-1.png)
- Example with 2 `origins`
- There are regional edge cached 
    - site between `origin` and `edge locations`
    - 12 of these
- 212 `Edge locations`
    - once a user  connects to the edge location all traffic goes over the AWS global network
- `Cloudfront` can push content to an edge location
    - if there is a cache miss on a user request at the `Edge Location` then it will go to the `Regional Edge Cache`, if there is a miss there then there will be an `origin fetch` to get the file from the origin
    - This will be cached along the way and will expire with the TTL
    - If you have dynamic objects it would be better to have a smaller TTL


![text](./images/route53/cfc-2.png)
- You can define a maximum TTL and default ttl
- Different file types can have different TTL
- Cna use headers to control the cache
- After expiration cloudfront will ensure it has the latest version

- `Path Patterns`
    - ex: any requests for *.jpg should be sent to `origin1`
        - the path pattern determines where toi send the request
    

![text](./images/route53/cfc-3.png)
- Can configure `CloudFront` to forward headers in request to the `origin`
    - Can forward all headers, a whitelist or only default headers
    - `Cloudfront` can then cache multiple versions of an object based on the values in one or more request headers


### CloudFront Signed URLs and OAI/OAC ###
![text](./images/route53/signed-url.png)
- `Signed Urls` provide more control over access to content, like giving a user a 1 time url for access
    - individual files
    - can specify beginning and expiration 
    - can specify IP addresses
    - should be used for individual files and clients that do not support cookies

- `Signed Cookies` - similar tyo `Signed Urls`
    - use when you don't want to change urls
    - **can provide access ot multiple restricted files**

![text](./images/route53/oai-oac.png)
- `Cloudfront Origin Access Identity`
    - A special type of user/rile that allows the `Cloudfront` distribution to access he custom origin `S3` bucket
    - restricts access to `OAI` only
    - only useful for `S3`
    - **Deprecated**
- `Cloudfront Origin Access`
    - recommended from AWS
    - like an OAI but with more use cases
    - Requires S3 Bucket policy that allows CloudFront service principle
    - restricts content so it can only be accessed by the cloudfront Distribution


### Cloudfront Cache and Behavior Settings ###
[see files here](./aws-saa-code-main/aws-cloudfront)
1. Create 3 `S3` buckets
    - pdf-bucket
    - jpeg-bucket
    - static website
2. Configure `S3` for the static website:
    - Enable public access
    - Configure as a static website
    - Add the index.html (when ready)
    - have to update the bucket policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Allow access",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:getObject",
            "Resource": "bucket name"
        }
    ]
}
```
3. Configure Amazon CloudFront:
    - Create a new CloudFront distribution
    - Add the static website as an origin (use website endpoint)
    - Disable caching
    - Add 2 more origins for the buckets containing the files and create/configure OAC
    - Configure cache behavior settings for each origin based on file type (PDF or JPG) and default going to the static website

### Cloudfront SSL/TLS and SNI  ###
![text](./images/route53/ssl-tls.png)

- To use `AWS Certificates Manager (ACM)` the CloudFront certificate must come from `us-east-1`
    - can bring your own certificate
- Default CF domain name can be changed using `CNAME` DNS record
- You can SSL secure S3 and `Customer Origins`
    - `S3 Origin` already has this configured and cannot be changed
    - `Origin` certs must be public
    - `Custom Origin` (ie: `EC2` or `ALB`)
        - `EC2` - can use 3rd party cert
        - `ALB` - can use `ACM` cert

- When a user issues a request it will be secured with SSl/TLS 
    - known as the `viewer protocol` - the protocol between the viewer and the user
        - ean encore HTTPS or allow HTTP as well, upgrade to HTTPS etc...

- `CloudFront Server Name Indication (SNI)`
    - allows you to have to separate TLS certificates corresponding to different domain names running on the same IP address in `CloudFront`
    - users can connect to different domain names but hit the same `Cloudfront Distributions`
        - knows this from the `host` header


### Lambda Edge ###
![text](./images/route53/lamda-edge.png)
- Allows you to run Node.js and Python functions to customize what is returned to the viewer
    - Cane be run at multiple points during the life cycle

### AWS Global Accelerator ###
![text](./images/route53/global-accelerator.png)

- Networking service that allows you to utilize AWS Global Network to send data to your applications
    - provides lower latency and more consistency then the regular internet

- 2 Users in the US
- If we have 2 endpoints and are using `Global Accelerator`, `Route 53` will return back 2 static anycast IP addresses that point to `Global Accelerator Endpoint`
    - when you use an alias record in `Route 53` you map the domain name to the domain name of the `Global Accelerator` (or other service) endpoint
        - need to resolve the DNS name of the `Global Accelerator` to its IP
        - either IP will take you any region that is active
            -  you should be directed to the region closest if there are no weights
    - Will eventually be direction ot an `AWS Edge Location` closest to the user
        - use `AWS global network` to connect to the Application Endpoints
            - uses the static anycast IPs
        - only use the public internet for the users to get from their device to the `edge location`
- If one site fails the request will be re-routed to the other endpoint
    - even if a user in the us has to go to `ap-southeast-2` since they are using the `AWS global network` the connection should be a lot faster and more reliable

## Block and File Storage ##
- https://digitalcloud.training/amazon-ebs/
- https://digitalcloud.training/amazon-efs/


### Block vs File vs Object Storage ###
![text](./images/ebs/block-storage.png)
- Block based storage = hard drives and SSD
    - HDD is a magnetic drives and spinning disk
        - Slower and cheaper than SSD
    - SSD - flash memory
        - newer and faster
        - more expensive
    - Create volumes
    - Can paritition the drive to different sizes


![text](./images/ebs/file.png)
- somewhere there is a block based storage system someone made a file system on top of (NTFS)
- Used network attaches storage server (NAS)
    - OS can interact with it just like the other storage
    - Connection is maintained

![text](./images/ebs/object.png)
- user uploads `objects` over HTTP rest API
    - ie: in console or rest api
        - very easy to integrate into applications
    - `S3` supports all file types
    - no defined hierarchy
    - massively scalable

![text](./images/ebs/comparison.png)


### Amazon EBS Deployment and Volume Types ###
![text](./images/ebs/deployment.png)
- `EBS volume` is provisioned and attached when launching an `EC2`
    - OS runs from here
- Data is in a single AZ
    - couple replicate to multiple copies but always int he same AZ
- Can connect multi Ec2 instances to an EBS volume using `EBS MultiAttach` as long as `EC2` is in the same `AZ`
    - Only avialable for `Nitro` based instances
    - Must be in same `AZ`
    - Max 16
    - Must be provisioned IOPS
- Can copy a snapshot of the volume to another `AZ`
![text](./images/ebs/types.png)
- GP SSD 
    - GP3 - newer
    - GP2 SSD - default for `EC2`
        - no multi attach
- Provisioned IOPS SSD
    - support multi-attach
    - good for low latency
    - IO2 Block Express - newer 
    - IO2 - deprecated in Nov 2023
    - IO1
        - must fast IOPS
        - only disk type that supports multi attach
    
![text](./images/ebs/hdd-types.png)
- HDD - cheaper
    - st1 and sc1
    - limited IOPS compared to ssd
    - save money for warehouse
    - no root capability

- EBS Persist independently of `EC2`
- Do not need to be attached to an instance
- Can attach a multiple volumes to a single instance
- **Must be in same AZ as `EC2`**
- `Root EBS volumes` are delete on `EC2` deletion by default
    - non boot volumes are not delete by default

### Amazon EBS Copying, Sharing and Encryption ###
![text](./images//ebs/sharing.png)
- `Snapshot` - point in time copy of the data on the volume
    - store in `S3`
        - since this is `regional` we can then create a `EBS Volume` from this in a different `AZ`
    - snapshots are incremental 
    - to free storage space, delete all old snapshots and just keep the latest
    - Can create `Amazon Machine Image (AMI)` from the snapshot 
![text](./images/ebs/ami.png)
- `Volume` can create a `snapshot`
- `Snapshot` can create an encrypted `snapshot`

- `Unencrypted Snapshot`
    - Can create encrypted volumes from 
    - cannot create an encrypted `AMI` just unencrypted
- `Encrypted Snapshot` 
    - can create an encrypted `AMI` from an encrypted snapshot
        - cannot share this publicly
        - must have the custom key

- `Encrypted AMI`
    - can create an `EC2` instance and change the encryption key
        - can change encryption state
    - Can create another encrypted `AMI` and change the key
        - change the `region`

- `Unencrypted AMI`
    - Can create `EC2`
        - change encryption station
        - change `AZ`




### Amazon EBS Snapshots and DLM ###
![text](./images/ebs/dlm.png)
- `Data LifeCycle Manager (DLM)`
    - can automate creating `Snapshots` and `AMI` for us
    - protect data by encoring regular backup schedule
    - creates standardized AMIs that can be refreshed at regular intervals
    - retain backups for compline and auditors
    - reduce storage cost by deleting old backups
    - created disaster recovery backup policies
        - backup data to isolated accounts 


### EC2 Instance Store Volumes ###
![text](./images/ebs/instance.png)
- Instance Store Volumes are local disks that are physically attached to host computer
    - extremely high performance
    - **ephemeral storage**
    - ideal for temporary storage that changes frequently
        - buffers, caches, scratch data
    - volume root devices are created from `AMI` templates stored on `S3`
    - Volumes cannot be detached and reattached


### Using RAID with EBS ###
![text](./images/ebs/raid.png)
- RAID - redundant array of independent disks
    - a way to take multiple disks and aggregate them together
    - **OS level not provided by AWS, must configure yourself**
    - RAID 0 and RAID 1 are potential options on EBS
    - RAID 5 and RAID 6 are not recommended
- RAID 0 - striping across multiple volumes
    - if any disk fails you lose your data
    - 2 + disks
    - better performance
- RAID 1 - mirroring
    - write the same data to two different volumes
    - redundancy and fault tolerance 
    - used double the amount of capacity 

### Amazon Elastic File System (EFS) ###
![text](./images/ebs/efs1.png)
- shared file system
- can connect instances from multiple `AZ`
- `Regional` File system
    - have mount targets in multiple `AZs`
    - appear as `ENI`
- Instacnes connect to the mount point in local AZ
- **Linux only**
- Uses NFS
- Can deployed in a single `AZ`  called `One Zone`
    - can connect from other `AZ` in the `Region`

- Data consistency - write operations for `Regional File Systems` are durably stored across AZ
- File Locking - NFS client apps can use NFS v4 file locking for read and write operations
- Storage classes
    - `EFS Standard` - uses SSds for low latency performance
    - `EFS Infrequent Access` - cost effective option
    - `EFS Archive` - even cheaper for less active data

- Durability
    - all storage classes offer 11 9s of durability


![text](./images/ebs/efs2.png)
- Can be replicated across regions 
    - Must create a mount point
    - the replica file system is read only
- Can connect on prem clients from outside of the cloud
    - must be linux and use NFS
    - typically will used a `Direct Connect`



- EFS replication - data is replicated across `Regions` for disaster recovery purposes
    - Recovery Point Objective (RPO)/Recovery Time Objective (RTO) in minutes
- Automatic backup - integrates with `AWS Backup` for automatic backups
    - `AWS BAckup` is a fully-managed service that makes it easy to centralize and automate data protection across AWS services, in the cloud, and on premises. Using this service, you can configure backup policies and monitor activity for your AWS resources in one place. It allows you to automate and consolidate backup tasks that were previously performed service-by-service, and removes the need to create custom scripts and manual processes. 

- Performance Options
    - `Provisioned Throughput` - specify level of throughput to be supported
    - `Bursting throughput` - scales with the amount of storage

- `AWS DataSync` - fast and simple way to securely sync existing files systems into `EFS`
    - Securely and efficiently copy files over internet or direct connect
    - copies file data and metadata 

### Amazon FSx ###
![text](./images/ebs/fsx.png)
- Fully managed third party file systems
    - `FSX` for windows file server
    - `FSX` for `lustre`
        - compute intensive


![text](./images/ebs/window.png)
- fully managed native windows file system
    - full support for many window protocols
- can be accessed by 1000s of instances
- High Availability - replicates data withing the `AZ`
- Multi AZ - create an active stand by file server in separate `AZ`
- Can have a managed microsoft `AZ` for auth
- Can access from an on premises client with a `VPN` or `Direct Connect Connection`

![text](./images/ebs/lustre.png)
- High performance workloads
    - machine learning
    - HPC
    - Video Processing
    - Financial modeling
    - Electronic design automation
- Works natively with `S3`
- `S3` objects are presented as files in your file system
    - can write results to `S3`
- Posix compliant file system
- Use for cloud bursting and data migration
- Can access from an on premises client with a `VPN` or `Direct Connect Connection`


### AWS Storage Gateway ###
![text](./images/ebs/storage.png)
- Allows you to connect on prem storage to AWS
- Deploy gateways into your on prem data centers
    - Allows to connect to storage on AWS
    - Encrypted in transit


![text](./images/ebs/file-gateway.png)
- Mounted with NFS or SMB in local data center
- local cache provides low latency
- virtual gateway appliance runs on some vm
- store files as objects in `S3`
    - multiple storage classes
    - application can talk in file system protocols while talking to `S3`

![text](./images/ebs/volume.png)
- ISCSI - block based only
- Cached Volume Mode - cache of most recently used data is stored on prem
- Stored Volume Mode - entire data set is stored on prem
    - backed up to `S3`
    - Snapshots are taken for backup

![text](./images/ebs/tape.png)
- Backup servers is on premises
- Connect to a virtual Tape Library
- Store in `S3`
    - once tapes are ejected from the backup app they can be stored in other storage classes like glacier and deep archive
- Used with popular backup software
    - NetBackup, Backup Exec, Veeam
- various sizes up to 2.5 TB
- Can have up to 1500 virtual tapes
    - max capacity of 1 PB
- All data is transferred between he gateway and AWS is encrypted with SSL
- All data stored in encrypted Server Side with `SSE-S3`
    - encryption keys managed by AWS


## Docker Containers and ECS ##
- https://digitalcloud.training/amazon-ecs-and-eks/

### Docker Containers and MicroServices ###
![text](./images/containers/virtualization.png)
- VMs ned an operating system which requires significant computational overhead

![text](./images/containers/docker.png)

- Containers on on a container engine anc can start very quick
    - they are resource efficient
    - container all code, settings and dependencies to run the application
    - isolated from other container
- Docker Hub os a cloud based registry service for sharing container images
- Containers are lightweight because they share the systems kerne;
- ideal for microservices and cloud native applications


![text](./images/containers/cna.png)
- Microservices architecture - structured collection of loosely coupled, independently deployable services
    -  each runs its out process
- Containers and Functions - code runs in docker containers or `Lambda` functions for isolation, elasticity and cost efficiency
- Can utilize message queues, apis,. auth, payment processor etc...


### Amazon Elastic Container Services (ECS) ###
![text](./images/containers/01-ecs-components.png)
- AWS Service to run Docker Container
- Must deploy and `ECS CLuster`
    - logical grouping of `tasks` or `services`
    - inside of a cluster you launch a `task`
    - `tasks` are created from `task definition`
    - `services` are used to maintain desired count of `tasks`
    - images can be stored in `Amazon Elastic Container Registry (ECR)`
- `ECS Container Instance` - the `EC2` that is running the container and have teh `ECS Container Agent`
    - this will join the instances the to `cluster`
- Serverless containers can run on `AWS ECS Fargate` with no overhead
- Fully managed container orchestration
- Docker Support
- Windows Container Support
- Integration with Load Balancer
    - defined as part of the service
- `Amazon ECS Anywhere` - enables ECS Control plane to manage on prem implementations
![text](./images/containers/02-ecs-features.png)
- `Cluster` - Logical Grouping of `tasks` or `services`
- `Container Instance` - `EC2` instance running the container agent
- `Task Definition` - blueprint for how the docker container should launch
- `Task` - a running container using settings from the `Task Definition`
- `Image` - docker image referenced in the `Task Definition`
- `Service` - defines long running `Tasks` 
    - can control `task` count with Auto Scaling and attach to a Load Balancer
![text](./images/containers/03-ecs-images.png)
- Containers are built from images defined in a docker file
    - Only Docker is supported on ECS
    - Images are stored in a registry such as DockerHub or `ECR`
        - `ECR` managed Docker registry
        - Supports private docker repositories with resource based permissions using `AWS IAM` in order to access repositories and images
        - Can use Docker CLI to push, pull and manage images
![text](./images/containers/04-ecs-task-definition.png)
- Required to run docker containers in `Amazon ECS`
- A text file in jSON format
    - describes 1 or more containers and max of 10
- Use docker images to launch container
![text](./images/containers/05-ecs-fargate.png)
- Managed `ECS` service
- Automatically provisions resources
- Provisions and manages compute
- Charged fro running `tasks`
- `EFS` Integration only
- Managed cluster optimization
- Limited control
![text](./images/containers/06-ecs-iam-roles.png)
- `ECS TASK` `IAM role` provides permissions the the container
- Container IInstance IAM role provides permissions to the host
- With Fargate the container instance role is replaced with the `Task Execution Role`

### Amazon ECS IAM Roles ###
![text](./images/containers/01-iam-roles.png)
- `Amazon ECS Container Instance IAM Role` - used by `EC2` and external instances to provide permissions to the container agent to call AWS APIs
- `Task IAM Role` - permissions granted in th IAM role are assumed by the containers running the task
- `Amazon ECS task execution IAM Role` - grants the `ECS` container and `Fargate` agents permissions to make AWS API calls
- `Amazon ECS infrastructure IAM role` - allows amazon to manage infrastructure resources 
- `ECS Anywhere IAM role` - when running on prem servers
- `Amazon ECS CodeDeploy IAM Role` - when code deploy is performing green/blue deployments
- `Amazon ECS EventBridge IAM Role` - `EventBridge` need permissions to use schedule tasks, rules and targets

![text](./images/containers/02-iam-roles.png)
- The container instance `IAM role` provides permissions to the host
- The `ECS Task` `IAM ROle` provided permissions to the container
- In `Fargate` IAM Task Roles cna de applied
![text](./images/containers/03-iam-roles.png)
- A container can only retrieve credentials from the IAM role that is defined in the `task definition` to which it belongs
- A container never has access to credentials that are intended for another container that belongs to another task
- When using `EC2` instances, `tasks` are not prevented from accessing credentials supplied to `IAM Instance role`

### Scaling Amazon ECS ###
![text](./images/containers/01-scaling.png)
- Two Types of Scaling
    1. service auto scaling
        - automatically adjusts the desired task count up or down using `Application Auto Scaling Service`
        - Supports target tracking, step and scheduled auto scaling policies
        - Ex: Metric reports CPU > 8-%
            - `CloudWatch` notifies `Application Auto Scaling`
            - `ECS` launched additional `task`
    2. cluster auto scaling
        - uses `Capacity Provider` to scale the number of `ECS Cluster` instances using `EC2 Auto Scaling`
        


![text](./images/containers/02-scaling.png)
- Scaling policies:
    - Target Tacking - increase or decrease the number of tasks that your service runs based on target value for a specific `CloudWatch` metric
    - Step Scaling - increase or decrease the number of tasks that your service runs in response to `CloudWatch` Alarms
        - Step Scaling is based on a set of scaling adjustments, know as step adjustments, 
        which vary based on the size of the alarm breach
    - Scheduled Scaling - increase or decrease the number of tasks that your service runs based on the date and time


![text](./images/containers/03-scaling.png)
- Cluster Auto Scaling:
    - `ASG` is linked to `ECS` using `Capacity Provider`
        - `Capacity Provider Reservation` metric measures the total percentage of cluster resources needed be all `ECS` workloads in the cluster
        - Managed Sacling - with an automatically created scaling policy on the `ASG`
        - Managed instance termination protection - enabled container aware termination of instance in the `ASG` when scale-in happens
    - CloudWatch notifies `ASG`
    - AWS Launches additional container instance




### Amazon ECS With ALB ###
![text](./images/containers/ecs-alb.png)
- Dynamic port is allocated on the host
- Each `task` is running on web service port 80
    - however we can only have 1 application running on port 80
    - must use the host port
- All connections to web services coming into HTTP listener 
- If applications running in private subnet we need a `NAT Gateway` in a public subnet and an entry in the route table for the private subnet
- The `Application Load Balancer` is container aware 
    - knows about the host port

### Launch Docker Containers on AWS Fargate ###
1. create a cluster
    - this will cause `cloudformation` to spin up a stack
2. Create a `task definition`
    - must assign a task role if you want the container to make API requests
    - Task Execution role - used by the container agent to make API requests
    - Chose image, ports and resources
    - Can configured storage, logs etc...
3a. Click run new `task` in the `cluster`
    - use the `task definition` just created 
    - will assign subnets, security groups etc..
OR 
3b.  Could create a `service`
    - adds load balancing and auto scaling to the `task definition`
    - will spin up the required amount of tasks

### Amazon Elastic Kubernetes Service (EKS) ###

![text](./images/containers/01-eks.png)
- Managed Kubernetes environment running in the cloud or on prem
- Used when you need to **standardize** container orchestration across multiple environments 
- Features:
    - Hybrid deployment - manage Kubernetes cluster across AWS and on prem
    - Batch Processing - run sequential or parallel batch workloads
    - Machine Learning - use KubeFlow with EKS to model your ML workflows and efficiently run distributed training jobs using the latest `EC2` GPU powered instances such as `Inferentia`
    - Web Applications -  scalable and available web applications across multiple `AZ`

- Supports Load Balancing with `ALB`, `NLB` and `CLB`
- Runs on `EC2`, `Fargate` or `AWS Outposts`
- Group of containers are kno as `Pods`
- Live in `EKS Cluster`

![text](./images/containers/02-eks.png)
- `Vertical Pod Autoscaler` - adjust the CPU and memory of pods to "right size" the applications
- `Horizontal Pod AutoScaler` - automatically scaled the number of pods in a deployment, replication controller or replica set based on resources CPU Utilization
- Supports 2 autoscaling products
    - Kubernetes Cluster Autoscaler
        - uses AWS `ASG`
    - Karpeneter open source autoscaling project
        - works directly with `EC2 Fleet`

- Load Balancing
    - support `NLB` and `ALB`
    - `AWS Load Balancer Controller` manages `AWS Elastic Load Balancers` for the cluster
        - Install the `Load Balancer Controller` using Helm V3 or later by applying a Kubernetes Manifest
        - Controller provisions the `ALB` when your create a Kubernetes Ingress
        - `NLB` when you create a Kubernetes service with type Load Balancer
    = 


![text](./images/containers/03-eks.png)
- `EKS Distro` is a distributions of Kubernetes with the same dependencies as `EKS`
- Allows you to manually run Kubernetes anywhere
- Includes binaries and containers of open-source Kubernetes etcd, networking and storage plugins
- Can securely access `EKS Distro` releases on GitHub or within AWS vias `S3` and `ECR`
- Alleviates the need t track updates, determine compatibility and standardize on common Kubernetes version across distributed teams
- You can create `EKS Distro` clusters in AWS on `EC2` or on prem
- 

### Amazon Elastic Container Registry (ECR) ###
![text](./images/containers/01-ecr.png)
- Fully Managed Container Registry
- Integrated with `ECS` and `EKS`
- Supports Open Container Initiative (OCI) and Docker Registry HTTP API
- Can user Docker Cli
- Can be accessed from Docker env in the cloud, on prem, or on your machine

- Container images and artifacts are stored in `S3`
- Can use namespaces to organize repos
    - Public repos allow everyone to access images
- Access control applies to private repos
    - `IAM Access control` - set of policies to define access to container images in private repos
    - `Resourced Based Policies` - access control down to the individual API Actions

![text](./images/containers/03-ecr.png)
- `Registry` - provided to each AWS Account, can create 1+ repos in a registry
- `Authorization token` - must authenticate with ECR as AWS user before can push and pull images
- `Repository` - contains images
- `Repository policy` - control access to your repos and images within them
- `Image` - push and pull container images to repo

- `LifeCyle Policies` - manage the lifecycle of the images in your repo
- `Image Scanning` - identify vulnerabilities in images
- `Cross Region and cross account replication` - replication of images 
- `Pull through cache rules` - cache repos in remote public registries in your private `ECR`
![text](./images/containers/04-ecr.png)
- Must have specific IAM permissions to push an image to the repo

![text](./images/containers/05-ecr.png)
1. authenticate
2. tag image
3. push image with docker cli

### AWS App Runner ###
![text](./images/containers/app-runner.png)
- fully manage service for deploying `containerized` web apps and apis
- PaaS solution with components managed , just bring code
    - could be from `ECR` or github etc...
- offers
    - Auto Scaling 
    - LB
    - Health Check 
    - Network
    - Observability
- Limited control



## Serverless Applications ##
- No need to manage any infrastructure at all
- Can build multi-layer applications
- Builds event driven architecture

- https://digitalcloud.training/aws-lambda/
- https://digitalcloud.training/amazon-api-gateway/
- https://digitalcloud.training/aws-application-integration/#amazon-simple-notification-service
- https://digitalcloud.training/aws-application-integration/#amazon-simple-queue-service-amazon-sqs
- https://digitalcloud.training/aws-application-integration/#amazon-simple-workflow-service-amazon-swf
- https://digitalcloud.training/aws-application-integration/#amazon-mq
- https://digitalcloud.training/aws-application-integration/#aws-step-functions
- https://digitalcloud.training/amazon-cloudwatch/


### Serverless Services and Event Driven Architecture ###
![text](./images/serverless/serverless.png)
1. User uploads a file through a `static website` hosted in `S3`
2. Lambda Serverless function processes the file
    - sends the ouput to an `S3` bucket
    - send a request to an `SQS` queue
3. `Lambd` pulls from the queue and publishes message to `SNS Topic` and `DynamoDB Table` 
    - Notification is sent uses `SNS` to email


- No instances to manage
- Do not need to provision hardware
- No management of operating system
- Capacity provisioning and patching is handled automatically
- Provides automatic scaling and high availability
- Can be very cheap
    - only pay for the exact compute that you use
    - pay more for more memory


### AWS Lambda ###
![text](./images/serverless/01-lambda.png)
- Developer uploads some code to `Lambda`
- Event occurs and triggers the `Lambda` function
- Code is executed
- Scales automatically
- Only pay for the compute you consume
- Benefits:
    - no servers to manage
    - continuous scaling
    - millisecond billing
    - integrates with almost all other AWS services

![text](./images/serverless/02-lambda.png)
- Primary use cases:
    - data processing, real time file or stream processing, serverless backends
- Invoked by:
    - Synchronous
        - CLI, SDK, API Gateway
        - Wait for the function to process the event and return a response
        - error handling happens on the client side
    - Asynchronous
        - `S3`, `SNS`, `CloudWatch` 
        - Event is queued for processing and response is returned immediately
        - `Lambda` retries up to 3 times
        - Processing must be idempotent
        - `Lambda` is triggered
    - Event Source Mapping:
        - `SQS`, `kineses Data Streams`, `DynamoDB Streams`
        - `Lambda` does the polling
        - Records are processed in order 
            - Except for `SQS` standard
![text](./images/serverless/03-lambda.png)
- Additional functions are initialized up to the `burst` or `account limit`
- If limit is exceeded throttling occurs with the error `429 Rate Exceeded` and a `TooManyRequestsException`
- Quotas:
    - 3000 - US West, East and Europe (Irland)
    - 1000 - Asian Pacific (Tokyo), Europe (Frankfurk), US East (Ohio)
    - 500 - other regions

- Max invocation time for 15 mins
    - is you have code that is required to run in a short period of time it is great
    - default timeout is 3 seconds

- Must ensure that the `Lambda` has the correct permissions to execute the required actions
    - the default permissions include access to `cloudWatch` 
- Can monitor invocations and runs tests from the console

### Application Integration Services Overview ###
![text](./images/serverless/services.png)
- Application integrations services sit in between layers of your application
- `SQS` - message queue - **decoupling**
    - for building distributed/decouple applications
- `SNS` - set up operate and send notifications from teh cloud
    - send email notification when `CloudWatch` alarm is triggered
- `Step Functions` - out of the box coordination of AWS Service components with visual workflow
    - order processing workflow
    - recommended over `SWS`
- `Simple Workflow Service` - need to support external processes or specialized execution logic
    - older version of step functions
    - human enabled workflows like an order fulfillment system
- `Amazon MQ` - queue based on Apache ActiveMQ or Rabbit MQ
    - need a message queue thats supports industry standard protocols or migrating queues to AWS
- `Amazon Kinesis` - collects, processes and analyzes data
    - collect data from IOT for later processes


### Amazon SQS ###
![text](./images/serverless/01-sqs.png)
![text](./images/serverless/02-sqs.png)
- Allows us to decouple the wen tier from the app tier to help alleviate overloading of the app tier
- Standard queue offers Best Effort Ordering
    - this not guaranteed order
    - unlimited number of transactions per second
    - At least once delivery
- FIFO - first in first out
    - 300 messages per second if not batched
    - Exactly once delivery
    - requires a `Message Group ID` and `Messages Deduplication ID`
    - `Message Group ID`
        - tag that specifies that a message belongs to a specific message group 
        - Messages that belong to the same group are guaranteed to be processed in a FIFO manner
    - `Message Deduplication ID` - token used for deduplication of messages within the deduplication interval

![text](./images/serverless/03-sqs.png)
- `Dead Letter Queue` - standard or FIFO
    - main task is handling message failures
    - sets aside a copy of the message into another queue
    - can analyze or process that message as required
    - no a queue type, just an extra Queue
    - Can specify it in the settings    


![text](./images/serverless/04-sqs.png)
![text](./images/serverless/05-sqs.png)
- `Delay Queue` - allows us to delay the visibility of the message for a certain period of time
    - becomes visible after the timeout
- Long Polling - retrieves messages from `SQS Queues` - waits for messages to arrive
    - can lower costs since we have to pay for API calls
        - reduce number of API operations from our consumer
    - can be enabled at the queue level or API level with the `WaitTimeSeconds` setting
    - 0 - 20 seconds
- Short Polling -  returns immediately even if the queue is empty



### Amazon SNS
![text](./images/serverless/01-sns.png)
- All about sending notifications
- pub/sub
- many Subscribers listen for notifications
1. Event Producer send one message to SNS Topic
2. Each subscriber will get the message

- Highly available, durable, secure, fully managed
- Provides `Topic` for high throughput, push based, many to many messaging
    - **PUSH BASED**
- Can fan out messages to a large number of endpoints
    - `SQS`
    - `AWS Lambda`  
    - HTTP/S Webhooks
    - Mobile push
    - SMS
    - Email

![text](./images/serverless/02-sns.png)
- multiple recipients can be group in `topics`
    - a topic i an `access point` for allow recipients to subscribe
- one topic can support deliveries to multiple endpoint types
- Simple APIs and easy integrations with web applications
- Flexible Message Delivery over multiple transport protocols

- Fan out - comes up in exam questions
    - can subscribe one or more `SQS` queues to an amazon `SNS` tropic
    - `SQS` manages the subscription and any necessary permissions
    - When you publish a message to a topic, `SNS` sends the message to every subscribed queue


### Simple Event Driven APP ###
![text](./images/serverless/event-driven.png)
- Add a message to a queue in the CLI
- `SQS` triggers `Lambda`
- Item is written to `DynamoDB Table` 
- `Cloudwatch logs ` tracks this



### AWS Step Functions

![text](./images/serverless/step-functions.png)
- Used to build distributed applications as a series of steps in a visual workflow
- Can quickly build and run state machines to executed the steps of your application
1. define the steps of the workflow using `JSON Based Amazon States Language`
    - can see what the looks like in the console
2. Start an execution to visualize and verify the steps of your application are operating as intended
    - operates and scales the steps of your application and underlying compute


### Create a State Machine ###


### Amazon EventBridge ###
![text](./images/serverless/01-event-bridge.png)
- Serverless event bus
    - can react to state changes in resources
- Used to be known as `CloudWatch Events`
- Sources publish events to the `EventBridge Bus`, some rules are applied and data is forwarded to targets
- Example:
    1. `EC2` instance is terminated
    2. `EventBridge Event Bus` gets the event
    3. A `rule` is processed
    4. `Rule` is sent to a target
    5. `SNS` sends an email


![text](./images/serverless/02-event-bridge.png)
- Define an event by creating an `Even Pattern`

- Example2:
    1. `CloudTrail` is a source
    2. `S3:PutBucket` api is used
    3. `EventBridge` receives this event
    4. `Rule` is applied
    5. `SNS` Sends notification

### Create Event Bus and Rule
![text](./images/serverless/event-bus.png)


### Amazon API Gateway ###
![text](./images/serverless/01-gateway.png)
- Used to create APIs to expose AWS services over an API
    - example: `EC2s`, `Lambda`, external services
- Supports Restful APIs and websocket APIs
- Can import Swagger/OpenAPI 3.0
- Can validate or modify request to forward it correctly

- Deployment types
 - `Edge-Optimized` - sits behind `CloudFront`
    - reduced latency for requests around th world
    - Uses `CloudFront Edge Locations` 
 - `Regional Endpoint` - reduced latency for requests from the same region
    - can include a CDN and Firewall
 - `Private Endpoint` - securely expose rest APIs only to other sevices within your `VPC` or connect via `direct connect`



![text](./images/serverless/02-gateway.png)
- Web APP connects to the Publish API
    - configure method requests and map them to the integrations
    - map the status codes, headers and payload received from the backend 
- For `Lambda` you cna have 
    - `Lambda` proxy integration
    - Custom integration
        - passes the request straight though
-  HTTP Endpoint
    - Proxy
    - Custom
- AWS Service action , only non proxy

![text](./images/serverless/03-gateway.png)
- Can add caching to api calls by provisioning the `Amazon API Gateway` cache
    - must specify its size
- Allows you to cache the endpoints response
- Can reduce number of calls to the backend and improve latency of requests to the API

- Throttling
    - sets limits on a steady-state rate and burst of requests submissions against all APIs in your account
        - Default limit in the steady state is 10,000 req/s
        - Max concurrent requests is 5000 across all APIs within an account
        - If you go over these limits you may get a 429
    - Upon catching exceptions the client and resubmit
![text](./images/serverless/04-gateway.png)
- Users connect to a specific api public endpoint with the API Key that is configured in the usage plan
    - Basic and premium users can have different limits, can determine who is who by the API Key
- Can also configure perm-method throttling limits




## Database and Analytics ##

### Database Types and Use Cases ###

![text](./images/db-and-analytics/01-db.png)
![text](./images/db-and-analytics/02-db.png)
![text](./images/db-and-analytics/03-db.png)
![text](./images/db-and-analytics/04-db.png)
![text](./images/db-and-analytics/05-db.png)

### Amazon Relational Database Service (RDS) ###
![text](./images/db-and-analytics/01-rds.png)
![text](./images/db-and-analytics/02-rds.png)
![text](./images/db-and-analytics/03-rds.png)

### Amazon RDS Backup and Recovery ###
![text](./images/db-and-analytics/backups.png)

### Create Amazon RDS Database ###

### Create a Read Replica ###

### Amazon RDS Security ###
![text](./images/db-and-analytics/01-security.png)
![text](./images/db-and-analytics/02-security.png)

### Create Encrypted Copy Of RDS Database ###

![text](./images/db-and-analytics/encryption.png)

### Amazon Aurora ###

![text](./images/db-and-analytics/aurora-1.png)
![text](./images/db-and-analytics/aurora-2.png)


### Amazon Aurora Deployment Operations ###
![text](./images/db-and-analytics/aurora-3.png)
![text](./images/db-and-analytics/aurora-4.png)
![text](./images/db-and-analytics/aurora-5.png)

### Amazon RDS Proxy ###
![text](./images/db-and-analytics/rds-proxy.png)



### Amazon Elasticache ###
![text](./images/db-and-analytics/elasticache-1.png)
![text](./images/db-and-analytics/elasticache-2.png)


### Scaling Elasticache ###

![text](./images/db-and-analytics/elasticache-3.png)
![text](./images/db-and-analytics/elasticache-4.png)

### Create Elasticache Cluster ###

### Amazon DynamoDB  ###

![text](./images/db-and-analytics/dynamo-1.png)
![text](./images/db-and-analytics/dynamo-2.png)
![text](./images/db-and-analytics/dynamo-3.png)

### Creating A DynamoDB Table ###

### DynamoDB Streams ###
![text](./images/db-and-analytics/dynamo-streams.png)

### DynamoDB Accelerator ###
![text](./images/db-and-analytics/dynamo-accelerator-1.png)
![text](./images/db-and-analytics/dynamo-accelerator-2.png)

### DynamoDB Global Tables ###
![text](./images/db-and-analytics/dynamo-global-tables.png)

### Enable Global Table ###

### Amazon RedShift ###
![text](./images/db-and-analytics/redshift-1.png)
![text](./images/db-and-analytics/redshift-2.png)
![text](./images/db-and-analytics/redshift-3.png)


### Amazon Elastic Map Reduce ###
![text](./images/db-and-analytics/emr.png)

### Amazon Kinesis ###
![text](./images/db-and-analytics/kinesis-1.png)
![text](./images/db-and-analytics/kinesis-2.png)
![text](./images/db-and-analytics/kinesis-3.png)
![text](./images/db-and-analytics/kinesis-4.png)

### Amazon Athena and AWS Glue ### 
- https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/

![text](./images/db-and-analytics/glue-1.png)
![text](./images/db-and-analytics/glue-2.png)
![text](./images/db-and-analytics/glue-3.png)

### Query S3 ALB Access Logs with Athena ### 


### Amazon OpenSearch Service (ElastiSearch)
![text](./images/db-and-analytics/elasticache-1.png)
![text](./images/db-and-analytics/elasticache-2.png)
![text](./images/db-and-analytics/elasticache-3.png)
![text](./images/db-and-analytics/elasticache-4.png)
![text](./images/db-and-analytics/elasticache-5.png)
![text](./images/db-and-analytics/elasticache-6.png)


### AWS Batch
![text](./images/db-and-analytics/aws-batch.png)

### Other Database Services 
![text](./images/db-and-analytics/other-db-1.png)
![text](./images/db-and-analytics/other-db-2.png)
![text](./images/db-and-analytics/other-db-3.png)

### Other Analytics Services ###
![text](./images/db-and-analytics/other-services-1.png)
![text](./images/db-and-analytics/other-services-2.png)
![text](./images/db-and-analytics/other-services-3.png)
![text](./images/db-and-analytics/other-services-4.png)
![text](./images/db-and-analytics/other-services-5.png)


## Deployment And Management ##
- https://digitalcloud.training/aws-cloudformation/
- https://digitalcloud.training/aws-elastic-beanstalk/
- https://digitalcloud.training/aws-config/
- https://digitalcloud.training/aws-resource-access-manager/
- https://digitalcloud.training/aws-systems-manager/
- https://digitalcloud.training/aws-opsworks/

### Infrastructure as Code With AWS CloudFormation ###
![text](./images/deployment/cloudformation.png)
- `AWS Cloudformation` - deploy infrastructure as code in a template file
    - Can be formatted in JSON or yml
    - `Cloudformation` will build the infra on AWS as defined in the template file
    - Good for reusability and consistency
- Components:
    - `Templates` - JSON or Yaml text file with instructions
    - `Stacks` - entire environment described by the template
        - if we delete the stack CloudFormation will delete all these resources for us
    - `StackSets` - extends `Stack` so they you can create, update and delete stacks across accounts or `regions`
    - `ChangeSets` - summary of proposed changes to the `stack` that will allow you to see how those changes might impact your existing resources before implementing them


### Creating and Updating Stacks 
- can upload a file and watch the events/resource tab to see app resources being created

- `!Ref` is a reference intrinsic function to dynamic select a resource
```yml
AWSTemplateFormatVersion: '2010-09-09'
Description: Create an EC2 instance with a security group for SSH access
Resources:
  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0440d3b780d96b29d
      InstanceType: t2.micro
      SecurityGroups:
        - !Ref InstanceSecurityGroup
```
- here we are referencing InstanceSecurityGroup

### Create Nested Stack using AWS CLI 
![text](./images/deployment/nested-stacks.png)
- can nest stacks inside of a nested stack to created a group of resources

### Platform as a Service with AWS Elastic Beanstalk
![text](./images/deployment/beanstalk-1.png)
- A platform as a service (PaaS)
- Everything is launched and managed by `Elastic BeanStalk`
    - creates an environment
- can upload the source code in a war/zip file

![text](./images/deployment/beanstalk-2.png)
- Supports many applications platforms
- Uses core AWS Services including `EC2`, `ECS`, `Auto Scaling`, `Load Balancing`
- Provides a UI to monitor and manage the health
- Managed platform updates deploy the latest version of software and patches
![text](./images/deployment/beanstalk-3.png)
- Layers:
    - Application
        - `environments`, env config, app versions
            - app version that have been deployed
            - resources are configured and provisioned by `BeanStalk`
            - comprised of all resources created by `BeanStalk`
        - can have multiple app version in an app
            - a specific reference to a section of code
            - typically will point to an `S3` bucket
    


![text](./images/deployment/beanstalk-4.png)
- `Web Servers` are standard apps listening to HTTP requests
- `Workers` - specialized applications that have a background processing task that listens for messages on an `Amazon SQS queue`
    - used for long running tasks


### Create an Elastic Beanstalk Service

### SSM Parameter Store
![text](./images/deployment/param-store.png)
- Provides secure, hierarchical storage for configuration data and secrets
- Highly scalable, available and durable 
- Store data such as passwords, db strings, license codes
- Store values in plain text or encrypted
- Reference values by using unique name that specified when creating the param
- **No native rotation of keys**
    - This is supported in `Secrets Manager`
    - Would have to do this explicitly with an external service like `Lambda`
- Free for standard, charged for advanced and higher throughput
- Allows hierarchical keys

### AWS Config
![text](./images/deployment/config-1.png)
- Viewing and managing config or AWS resources
    - often in compliance scenarios

- Evaluate your AWS Resource configurations for desired settings
- Get a snapshot of the current configurations of resource associated with AWS account
- Retrieve configs or resources in your account
- Retrieve historical configs of one or more resources
- Receive notification whenever a resource is created, modified or deleted
- View relationships between resources



- Example:
    1. `Config` evulates the config against desired configs
    2. changes occur and information is sent to `Aws Config`
        - recorded info in `S3`
    3. Send notification via `SNS Topic`
    4. Alert via cloudwatch events (now `EventBridge`)
    5. Can uses `Systems Manager` to automatically remediate unwanted state


![text](./images/deployment/config-2.png)
- Example AWS Managed Rules that can be found in config


### SSM Automation and Config Rules 

### AWS Secrets Manager
![text](./images/deployment/secrets-manager.png)
- Stores and rotates secrets safely without need for code deployments
- Allows for automatic rotation of the secrets
- Built in for:
    - `Amazon RDS` - 
    - `Amazon RedShift` - 
    - `Amazon Document DB` -
    - `Amazon ECS`
    - Other services you will need to rotate with  a `Lambda` function
- Charges per secret 
- String or binary, always encrypted
- 64kb

### Ops Works
![text](./images/deployment/ops-works.png)
- AWS OpsWorks is a configuration management service that provides managed instances of these two open-source tools (Chef and Puppet).

### AWS Resource Manager 
- https://docs.aws.amazon.com/ram/latest/userguide/shareable.html
![text](./images/deployment/ram.png)

- Shares resources within or across accounts, like subnets, dbs and resource groups :
    - Across accounts
    - Within `AWS Organizations`
    - `IAM Roles` and `IAM users`
- Can be created with:
    - `RAM` console or APIs
    - AWS CLI
    - AWS SDK



### Share a Subnet Across Accounts

### RPO, RTO, DR Strategies
- https://aws.amazon.com/blogs/architecture/disaster-recovery-dr-architecture-on-aws-part-i-strategies-for-recovery-in-the-cloud/

![text](./images/deployment/strategy-1.png)
![text](./images/deployment/strategy-2.png)
- Recovery Point Objective (RPO)
    - measurement of the amount of data that can be acceptably lost
    - measured in seconds, minutes or hours
    - Examples:
        - Milliseconds to Seconds require  Synchronous Replication
        - Seconds to Minutes require Asynchronous Replication
        - Minutes to hours require Snapshots and cloud backups
        - Hours to Days require offsite, traditional and tape backups
- Recovery Time Objective
    - How much time it takes to restore from disaster
    - measured in seconds, minutes or hours
    - Examples:
        - Milliseconds to Seconds require Fault Tolerance
        - Seconds to Minutes require high availability, load balancing and auto scaling
        - Minutes to hours require cross-site recovery(cloud), automated recovery
        - Hours to Days require Cross site recovery (cloud/on-prem), manual recovery

![text](./images/deployment/strategy-3.png)
- Strategies
    - `Backup and Restore` - low priority, provision/restore after event, $
    - `Pilot Light` - data replicated, services are idle/off, resources activated after the event, $$
    - `Warm Standby` - Minimum resources always running, business critical workloads, scale up after, $$$
    - `Multi-site active/active ` - Zero downtime, nero zero data loss, mission critical workloads, $$$$

## Monitoring, Auditing and Logging 
- https://digitalcloud.training/amazon-cloudwatch/
- https://digitalcloud.training/aws-cloudtrail/

### Amazon CloudWatch Overview
![text](./images/monitoring/cw-1.png)
- `CloudWatch` is a performance monitoring tool, can trigger alarms, log collection and automated actions
    - Use Cases:
        - Collect performance metrics from AWS and on prem systems
        - Automate responses to operational changes
        - Improve operational performance and resource optimization
        - Derive actionable insights from logs
        - Get operational visibility and insight
    - Core Features
        - `CloudWatch Metrics`
            - services send time-ordered data points 
            - Metrics sent every 5 mins by default for `Ec2`
                - sends every 1 min with `Detailed EC2 Monitoring`
                - `CloudWatch detailed` monitoring is enabled by default when creating launch configurations through the CLI.
            - unified `CloudWatch Agent` sends system level metrics for `EC2` and on prem servers
                - **include memory and disk usage (need the agent)**
                - collects log files and metrics together
            - can publish custom metrics from CLI or API
                - 2 Resolutions
                    - Standard  - data to the minute
                        - default resolution
                    - High  - data to 1 second
        - `CloudWatch Alarms` 
            - monitor metics and initiate actions
            - example - auto scaling group responding to amount of load
            - Types
                - `Metric` - performs one or more actions based on the alarm
                - `Composite` - uses a rule expression and takes into account multiple alarms
            - States
                - OK
                - ALARM - outside threshold
                - INSUFFICIENT_DATA - does not know enough
        - `CloudWatch Logs`
            - centralized collection of system application logs
        - `CloudWatch Events`
            - stream of system event describing changes to AWS resources
                - can trigger actions 
            - now `EventBridge`
    

![text](./images/monitoring/cw-2.png)
![text](./images/monitoring/cw-3.png)

### Create A custom Metric and Alarm 
- see [steps here](./aws-saa-code-main/amazon-cloudwatch/custom-cloudwatch-metrics.md)


### Amazon Cloudwatch Logs
![text](./images/monitoring/cw-logs.png)
- gather application and system logs in `CloudWatch`
- Defined expiration policies and `KMS` encryption
- Send to: 
    - `S3`
    - `Kineses Data Streams`
    - `Kineses Data FireHose`
- `Unified Cloudwatch Agent` is what is actually used to send data 
    - Need permissions to actually allow sending data to `CloudWatch`
- Can export information to destinations like `ElasticSearch`

### The unified CloudWatch Agent
- https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html

![text](./images/monitoring/cw-agent.png)
- Collect internal System Level metics from `EC2` instances across OS
    - Also on prem
- Can retrieve custom metrics
    - must use `StatsD` or `collectd` protocol
- Collect logs from `EC2` instances or  on prem (Windows/Linux)
- **Agent must be installed on the server**
    - logs files are sent in real time so wont lose the metrics when instance is terminated
    - Can be installed on:
        - `EC2`
        - On Prem
        - Linux, Windows Server or MacOS
            - Collects a ton of metrics for Linux and MacOS that is not in standard metrics
                - Disk data, memory data, 
            - Data from Windows Performance Monitor
    


### AWS CloudTrail

![text](./images/monitoring/cloud-trail-1.png)
- logs API Activity for auditing
    - ie: when user or automated tasks take action
- Retained for 90 days by default
- `CloudTrail Trail` logs any events to `S3` for indefinite retention
    - Can send notification via `SNS` when a log file irs published
    - Can enabled log file integrity validation
        - ensures logs have not been modified

- `Trail` can be within 1 `Region` or all `Regions`
- `CloudWatch Events` can be triggered based on API calls to `CloudTrail`
    - now `EventBridge`
- Events can be streamed to `CloudWatch` Logs



![text](./images/monitoring/cloud-trail-2.png)
- Event Types:
    - Management Events - provide information about manage operations that are performed on resources in your AWS Account
        - Launch an `EC2`, create  a DB or `Lambda` etc...
        - Stores who, when, what 
    - Data Events - provide information about resource operations performed on or in a resource
        - provides more detail
    - Insight Events - identify and respond to unusual activity associated with write API calls continuous analyzing `CloudTrail` management



### Create A Trail In AWS CloudTrail


### AWS EventBridge (Refresher)
![text](./images/monitoring/eventbridge.png)
- Used to be know as `CloudWatch Events`
- Has been separated out into separate console
- Data is sent from events in the sources to the `EventBridge Event Bus`
    - Then rules determine what to do with the data being received
    - decide which `target` receives the info

### Create EventBridge Rule for CloudTrail API Calls

### Metrics and Analysis Tracking 
![text](./images/monitoring/metrics-1.png)
- Use X_Ray to visualize components of your application, identify performance bottlenecks and troubleshoot request that resulted in an error
- Service send trace data and `X-Ray` processes the data to generate service mpa and searchable trace summaries

- `AWS X-Ray` - used with applications running on `EC2` `ECS`, `Lambda` and `Elastic Beanstalk` 
    - must integrate with `X-Ray SDK` with your application and install the `X-Ray Agent`
        - Agent is a software application that gathers raw segment data and relays it to the `X-Ray` Service
        - SDK captures metadata for request made to MySQL, Postgres and `DynamoDB`, as well as `SQS` and `SNS`

![text](./images/monitoring/metrics-2.png)
- `AWS Managed Prometheus` - 
    - Prometheus is an open-source monitoring system and time series database
    - Uses PromQL to monitor and alert the performance of containerized workloads
    - Scales the ingestion, storage, alerting and querying of operational metrics as workloads grow or shrink
    - Integrated with `EKS`, `ECS` and `AWS Distro for OpenTelemetry`
![text](./images/monitoring/metrics-3.png)
- Open source analytics monitoring solution for databases
- Highly scalable, available and aws managed
- Provides interactive data visualization for your monitoring and operation data
- Visualize analyze and alarm on your metrics. logs and traces collection from multiple data sources
- Integrates with `AWS SSO and SAML`
- Can visualize the data from `X-Ray`


## Security In the Cloud 
- https://digitalcloud.training/aws-iam/
- https://digitalcloud.training/amazon-cognito/
- https://digitalcloud.training/aws-directory-services/
- https://digitalcloud.training/aws-kms/
- https://digitalcloud.training/aws-cloudhsm/
- https://digitalcloud.training/aws-waf-shield/

### AWS Directory Service
![text](./images/security/ad-1.png)
- Provides a few different options
- `AWS Managed Microsoft AD` running on Windows Server
    - can have a pair of highly available Windows Domain Controllers
        - one or two wat trust relationship
    - can connect `EC2` linux and Windows to the domain controller
    - Can connect Azure and Office 365 for federated identities
    - example apps and services that support auth
    - Can apply group policies, you SSO and enable MFA using radius
    - Best choice if you have more than 5000 users or need to connect another domain using a trust relationship
    - Can perform schema extensions
    - Can setup trust relationships with On Prem AD
        - on prem user and group can access resources in either domain using SSO
        - requires a vpn or `direct connect`
    - Can be use as a standalone AD in the AWS cloud
![text](./images/security/ad-2.png)
- AD Connector 
    - when you have an existing AD on a corporate env
        - self managed
    - Connect over `VPN` or Direct `Connect`
    - Sign into AWS Applications with AD credential 
        - federated AWS sign in
        - Maps credentials to IAM roles
    - Can seamlessly join Windows `EC2` instances to an on prem AD Domain
    - Redirects requests to on prem AD 
    - Small - 500 users
    - Large - 5000 users
    - Allows login to Management Console with local AD credentials


![text](./images/security/ad-3.png)
- Inexpensive AD compatible service with common directory features
- Stand alone fully managed, directory in AWS Cloud
- Simple AD is generally the least expensive option
- Best choice for less than 5000 users and do not need advanced AD  features
- Features include:
    - Managed user accounts/groups
    - apply group policy
    - Kerberos Based - SSO
    - Supports joining Linux or Windows `EC2` instances


### Identity Providers and Federation 
![text](./images/security/id-1.png)
- AD is an LDAP identity Store
- Active Directory Federation Services is an `Identity Provider`
    - idp authenticates the users
    - idp sends client SAML Assertion
    - APP Calls `sts:AssumeRoleWithSAML` 
    - `AWS Security Token Service (STS)` returns temporary credentials
    - User is not allowed to use credentials to access `DynamoDB`
- Open ID Connect (OIDC)
    - Social Identity Providers 
        - apple, google, facebook etc...
    - App calls `sts:AssumeRoleWithWebIdentity` 
    - `AWS Security Token Service (STS)` returns temporary credentials
    - User is not allowed to use credentials to access `DynamoDB`
    - AWS recommend you use `AWS Cognito` rather than `Web ID Federation`


![text](./images/security/id-2.png)
- `IAM Identity Center` - sources can be Identity Center directory, Active Directory, and standard providers using SAML 2.0
    - Successor to `AWS SSO`
    - Enables centralized permissions management and SSO
    - Built in SSO with many common applications
    - Integration with `Organizations` 

- `AWS Cognito`
    - Used with web and mobile app configurations 
    - allows Social Identity Providers
    - The app calls the `Cognito User Pool` with Social ID provider ifo
    - A Token is generated with security credentials and returns a JWT
    - App calls `Cognito Identity pool` which calls the `AWS Secure Token Service` `sts:AssumeRoleWithWebIdentity` 
    - Token is exchanged for temporary security credentials
    - User is not allowed to use credentials to access `DynamoDB`

### IAM Identity Center in Action

### Amazon Cognito
![text](./images/security/cognito-1.png)
- Used for sign in/up functionality to web and mobile applications
- `Cognito User Pool` - directory for managing sign in and sign up for mobile applications
    - where Identities are stored or we federate to a identity provider
- Acts as an Identity broker between the Identity Provider and AWS
    - Uses JWT to pass around to confirm authentication/authorization
        - `Lambda` authorizer accepts and processes the JWT


- `Cognito Identity pool` - are used to obtain temporary limited privilege credentials for AWS Services
    -  use `AWS STS`
    - IAM role is assumed providing access to AWS Services
![text](./images/security/cognito-2.png)
1. Authenticate and get JWT
2. Exchange JWT for AWS credentials
3. Access AWS service with temporary credentials

### Encryption Primer 
![text](./images/security/encryption-1.png)
- `S3` encrypts the objet as it is written in the bucket
- Data is protected by SSL/TLS
![text](./images/security/encryption-2.png)

### AWS Key Management Service (KMS)
- **Multi-tenant AWS Service**
![text](./images/security/kms-1.png)
- Create and manage symmetric and asymmetric encryption keys
- The `KMS Keys` are protected by HSM
- Developer create customer managed `KMS` keys

- `KMS Keys` are primary resoruces in `AWS KMS`
- Used to be know as `customer master keys`
- `KMS` also contains the key material used to encrypt and decrypt data
- By default `AWS KMS` create the key material for a `KMS key`
    - You can import your own key material
    - A `KMS key` can encrypt data up to 4kb in size
    - can generate, encrypt and encrypt `Data Encryption Keys`



![text](./images/security/kms-2.png)

- External Key Store
    - Keys can be stored outside of AWS to meet regulator requirements
    - Can create a `KMS Key` in an `AWS KMS` external key store
    - All keys are generated and stored in an external key manager
    - When using XKS, key material never leaves your `HSM`
- Custom key store
    - can create `KMS keys` in `AWS CloudHSM` 
    - All keys are generated adn stored in `AWS CloudHSM` cluster that you own and manage
    - Cryptographic operations are performed solely in the AWS CloudHSM cluster 
    - Custom key stores are not available for asymmetric `KMS Keys`

- Manged KMS Keys
    - created, managed and used on your behalf by AWS Service that is integrated with `KMS`
    - Cannot manage these KMS keys, rotate them or change their key policies
    - Cannot use AWS managed `KMS Keys` in cryptographic operations directly, the service that create them uses them on your behalf

![text](./images/security/kms-3.png)
- `Data encryption Keys` 
    - encryption keys that you can use to encrypt large amounts of data
    - can use `AWS KMS` to generate, encrypt, and decrypt data keys
    - `AWS KMS` does not store, manage, or track you data keys, or perform cryptographic operations with data keys
    - You must use and manage data keys outside of `KMS`
- You cannot enable or disable key rotation for AWS owned keys
- Automatic key rotation supported only on symmetric encryption `KMS` keys with key material that `AWS KMS` generates


![text](./images/security/kms-4.png)
- Automatic rotation generates new key material every year
    - optional for `customer managed keys`
- Rotation only changes the key material used for encryption the KMS key remains the same

- With automatic key rotation:
    - Properties of the `KMS Key` including the ID, ARN, Region, Polices and permission do not change when the key is rotated
    - You do not need to change applications or aliases that refer to the key ID or key ARN of the KMS key
    - After you enable key rotation, `AWS KMS` rotates the `KMS Key` automatically every year
- Automatic Key rotation is not support for (can rotate them manually):
    - Asymmetric `KMS Keys`
    - `HMAC KMS Keys`
    - `KMS keys` in custom key stores
    - `KMS keys` with imported key material



![text](./images/security/kms-5.png)

- Manual rotation is created a new `KMS Key` with a different `key ID`
    - Must update your applications with the new `key ID` 
    - can use an `alias` to represent the `KMS key` so you don't modify your application code
        - the alias is associated with the new `KMS Key`   

- `Key polices` define management and usage permissions from `KMS keys`
    -  defines the administrative actions that are permitted for the key administrator


![text](./images/security/kms-6.png)
- Multiple policy statements can be combined to specify separate administrative and usage permissions 
- `Key policy` defines teh cryptographic actions for encrypting and decrypting data with the KMS key
- Permissions can be specified for delegating use of the key to `AWS Services`
    - grants useful or temporary permissions as they can be used without modifying `key policies` or `IAM policies`


![text](./images/security/kms-7.png)
- To share snapshots with another account you must specify `decrypt` and `CreateGrant` permissions
- The `kms:ViaService` condition key can be used to limit key usage to specific AWS services
- Cryptographic erasure means removing the ability to decrypt data can be achieved when using imported key material and deleting key material
- You must use the `DeleteImportKeyMaterial` API to remove key material
    - makes it impossible to decrypt this data 
- An `InvalidKeyID` exception when using `SSM Parameter Store` indicated the `KMS Key` is not enabled
- 

### AWS CloudHSM 
![text](./images/security/cloud-hsm.png)
- `Custom key store`
- Cloud based hardware security module
    - dedicated hardware device
    - **single tenancy**
- Generate and use your own encryption key
- runs in `VPC`
- Uses FIPS 140-2 level 3 validated HSM
- Managed service and automatically scales
- Retain control of your encryption keys
    - you control access
    - AWS has no visibility of yur encryption keys
- Use cases:
    - Offload SSL/TLS from web servers
    - Protect private keys for issuing certificate authority
    - Store the master key for Oracle DB transparent Data Encryption
    - Custom key store for `AWS KMS` - retain control of the `HSM` that protects mater keys
![text](./images/security/acm-1.png)

### AWS Certificate Manager (ACM)
![text](./images/security/acm-2.png)
- Create store, renew SSL/TLS X.509 certs
- Single domains, multiple domain names and wildcards
- Integrates with several AWS services


- Public certificates are signed by `AWS Public Certificate Authority`
    - can create a Privet CA
    - can issue private certs
    - can import certs from third party issuers

### SSL/TLS Certificate ACM

### AWS Web Application Firewall (WAF)
![text](./images/security/waf-1.png)
- Web Application firewall
- Create rules to filter web traffic based on conditions that include IP Addresses, HTTP headers and body, custom URIs
- Makes it easy to create rules to block common web exploits like SQL Injection and XSS


![text](./images/security/waf-2.png)
- `Web ACL` - Web Access Control Lis used to protect a set or resources
- `Rules` - contains a statement that defined the inspection, criteria and action to take if a web request meets the criteria
- `Rule groups` - can use rules individually or in reusable rule groups
- `IP Sets` - IP Set provide a collection of IPs and IP ranges you want to use together in a rule statement 
- `Regex Patter set` - provides a collection fo regex to use in a rule statement

![text](./images/security/waf-3.png)
- `Rule Action` tells `WAF` what to do with a wen request when it matches the criteria in a rule
    - Count - counts the request but doesn't determine whether to allow of block
        - `WAF`continues processing the remaining rules in the ACL
    - Allow - allows request to be forward to AWS resource
    - Block - block request from resource
        - returns 403
- Match statements compare the web request or its origin against conditions you provide
    - Geographic
    - IP Set
    - Regex
    - Size 
    - SQLi Attack
    - String Match
    - XSS

### Amazon inspector 
![text](./images/security/inspector-1.png)
- Runs assessments to check for security exposures and vulnerabilities in `EC2`
    - can be configured to run on a scheduled
    - Agent must be installed on `EC2` for hosts assessments
        - Network Assessments do no require an agent

- Network Assessments
    - Asses network config analysts to check for ports reachable from outside the `VPC`
    - If `Inspector agent` is installed on `EC2` instances, the assessment also find processes reachable on ports
    - Price based on number of instance assessments

![text](./images/security/inspector-2.png)

- Host Assessments
    - Asses vulnerable software (CVE), host hardening (CIS Benchmarks) and security best practices
    - Requires an agent
    - Price based on number on instance assessments

### Amazon Macie 
![text](./images/security/macie.png)
- Fully managed data security and privacy service
- Uses ML and pattern matching to discover, monitor adn help you to protect your sensitive data on `S3`
- Enabled security compliance and preventative security
    - Identities variety of data types including PII, PHI, regulatory docs, API and Secret Keys
    - Identify changes to policy and ACL
    - Continuous monitoring the security posture of `S3`
    - Generate security findings that you can view in `Macie` console, `AWS Security Hub` or `Amazon Event Bridge`
    - Manage multiple AWS accounts using `AWS Organizations`

1. Automatically discovers `S3` buckets
2. Analyzes buckets using ML
3. Sends findings to `CloudWatch Events`



### AWS Guard Duty
![text](./images/security/guard-duty.png)
- Intelligent threat detections service
- Detects account, instance, bucket compromise and malicious reconnaissance
- Continuous Monitoring for events across:
    - `AWS Cloudtrail Manage Events`
    - `AWS Cloudtrail S3 Data Events`
    - `Amazon VPC Flow logs`
    - `DNS Los`


### AWS Shield
![text](./images/security/shield.png)
- Managed DDOS protection service
- Safeguards web applications running on AWS with always on detection and automatic inline mitigations
- Helps minimize application downtime and latency
- Two tiers
    - standard - no cost
    - advanced - 3k per month and 1 year commitment
- Integrated with `Amazon CloudFront`
    - standard included by default

### Defense In Depth 
![text](./images/security/did.png)
- Practice of implementing security at multiple later of the infrastructure
- `KMS` - used to encrypt ebs volumes
- `Inspector` - check for vulnerabilities and send a notification if one is found
- `AWS WAF` - Protect against common exploits
- `AWS Shield` - DDOS protections
- `ACM` - protect our ALB with SSL




## Migration and Transfer 
- Many tools available to migrate into and out of AWS 
- https://digitalcloud.training/aws-migration-services/

### AWS Migration Tool Overview
### AWS Application Discovery Service 
![text](./images/migration/tools-1.png)
- Could be migrating into AWS from a corporate/on prem data center
    - have dbs, servers, NAS
- Connected with `Direct Connect`, `VPn` or the Internet
- `Application Discovery Service`    
    - collects data about on prem resources
- `AWS Application Migration Service` - can migrate servers into `EC2`
- `AWS Database Migration Service` - Migrates DBs into `RDS`
- `AWS DataSync` - migrates into `EFS`
- `AWS Migration Hub` - monitors migration that use AWS Services or Partner Tools


![text](./images/migration/tools-2.png)
- `Application Discovery Service (ADS)` 
    - can collect metadata such as hostnames, ip addresses, macs 
    - also resources including CPU, network, memory etc...
    - can use agent based discover for windows and linux OS
    - can be saved to `S3` and queried with `Athena` or `QuickSite`
- `Discovery Connector` - per center
    - VmWare allows an agentless discovery
    - Static Configs
    - VM utilization metrics
- `Discover Agent` - per server
    - other OS and agents not using VMWare
    - data forwarded to `ADS`
        - Static Configs
        - Time Series Performance
        - Network I/O
        - Running process
    


### Database Migration Service (DMS)
![text](./images/migration/dms-1.png)
![text](./images/migration/dms-2.png)

- USed for migrating databases
    - Can use `Schema Conversion Tool` for heterogenous migrations
        - Heterogenous = going from 1 db type to another
        - Homogenous = going from same db type to same db type
- Use Cases:
    - cloud to cloud
    - on prem to cloud
    - Development and Test 
    - Database Consolidation - consolidate multiple db into a single db
    - Continuous Data Replication - use fro single source multi-target or multi-source single target

### AWS Application Migration Service (MGN)
![text](./images/migration/mgn-1.png)
![text](./images/migration/mgn-2.png)

- Used for migrating servers
    - The agent-based replication is recommended where possible
        - supports continuous data protection
    - Can provide and agentless snapshot based replication with `AWS MGN vCenter Client` 
        - not recommended
    - Provides automated incremental and schedule migrations
    - can automate actions in conjunction with `Lambda`
    - `EC2 Instances` are launched from launch templates
- Used for **lift and shift** rehost of applications to AWS
    - continuos block level replication
        - cut over windows in minutes
    - Old Service was the `Server Migration Service (SMS)` which used cut over windows in hours
- Entire application group cna be migrated in a `migration wave`
    - could also use `CloudFormation`
- Can migrate both virtual and Physical Servers

### AWS DataSync
![text](./images/migration/datasync.png)

- Used for mirgating data
    - Can be shared file system or Object Bases Storage
- `DataSync` has a software agent that connects to the software system
    - can migrate to `S3` `FSX` or `EFS`
        - Supports multiple `FSX` services
    - Can transfer data between NFS, SMB, Hadoop, Sle manage object Storage, `Snowcone`, `S3` EFS and FSX
    - all data is encrypted in transit
    - supports schedule and automated transfers
- `AWS Snowcone` - a DataSync Agent is installed on the snowcone
    - Can transfer this data to AWS Services
- `Amazon S3 On Outpost` allows this awa well




### AWS Snowball Family 
![text](./images/migration/snowball-1.png)
![text](./images/migration/snowball-2.png)

- Physical, tamper resistant, encrypted devices
    - 256 bit encryption
        - manage with `AWS KMS` in tamper resistant `Trusted Platform Modules (TPM)`
- Client software is installed on your computer
- `Snowball` and `SnowMobile` are used for migrating large volumes of data to AWS
    - 10s - 100s of terabytes, data needs to be transferred quickly but do not have alot of bandwidth
    - Snowball 
        - 80TB or 50TB - petabyte scale
    - SnowMobile - 100 PB per snowmobile
        - exabyte scale
- `SnowBall Edge Compute Optimized`
    - provides object storage and option GPU
    - used for data collection, ML and processing, storage at the edge (factory, remote area etc...)
- `SnowBall Edge Storage Optimized`
    - provides block storage and `S3` compatible storage
    - Uses for local storage and large scale data transfer
- `Snowcone`
    - Small device used for ege computing, storage and data transfer
    - Can transfer data offline or online with `DataSync Agent`

- Optimization of data transfer
    1. Uses latest `snowball` client
    2. Batch Small files together
    3. Perform multiple copy operations at one time
    4. Copy from multiple work stations
    5. Transfer directions not files

- Use Cases
    - Cloud Data migration
    - Content Distribution
    - Tactical Edge Computing
    - machine Learning on device
    - Manufacturing - data collection and analysis in the factory
    - Remote locations with simple data 
        - pre-processing, tagging compression etc..


### The 7Rs of Migration 
![text](./images/migration/r-1.png)
![text](./images/migration/r-2.png)

- Refactor - completely re-architect the application to a cloud-native serverless architecture 
    - use cloud native architecture such as serverless functions or containers
    - migrate databases to managed or severless NoSqlDB
    - could be a decent amount of work
- Replatform
    - Move MySQl db to `RDS` and linux server to `Elastic BeanStalk`
        - may need some code update, db update etc...
        - can use `Database Migration Service` and `Schema Migration Tool`
- Repurchase - uses a different solution
- Rehost
    - what AWS terms as lift and shift 
        - move application from your host an `Ec2` host etc.. with `AWS Application Migration Service`
            - need `AWS Replication Agent` installed on servers if not VMWare
    - At some point we perform the final sync and cut over the instance
- Relocate
    - lift and shift the server
    - no os or application changes
- Retain - do nothing for now
- Retire - done with application, get rid of it


## Web, Mobile, ML, Cost Management

### AWS Amplify and AppSync
![text](./images/mobile/amplify.png)
- `Amplify` - tools and features for building full stack applications
    - build wbe and mobile backends and web frontend UIs
    - `AWS Amplify Studio` - a visual interface for building web and mobile apps
        - uses visual interface to define a data model, user auth, and file storage
        - easily add `AWS Services` not available within `Amplify Studio` using `AWS Cloud Development Kit`
        - Connect mobile and web apps using Amplify libraries for iOS, android, Flutter, React Native and web
    - `Amplify Hosting` is fully managed CI/CD and hosting service for fast, secure and reliable static and server side rendered apps
![text](./images/mobile/app-sync-1.png)
- `AppSync` automatically scaled a GraphQL API execution engine up and down to meet API requests
    - Fully managed service for GraphQL
    - Allows you to subscribe over a websocket connection and bundle many AWS Servies in 1 communication channel
        - ie: `Lambda`, `DynamoDB`, `Aurora`, Http etc...
    - lets you specify which portions of you data or you data should be available
    - Supports `AWS Lambda`, `DynamoDB` and `ElasticSearch`
    - Server-side data caching capabilities reduce the need to directly access data sources
    - Fully managed and eliminates the operational overhead of managing clusters

![text](./images/mobile/app-sync-2.png)
- `Amplify` is used to build and host the WebStore application and created backend services
- `AppSync` create a unified API layer for integrating microservices

### AWS Device Farm

- application testing service used for web and mobile apps
    - can automate testing
- can see what application looks like across different device and browsers
    - can also test access and interaction

### AWS Machine Learning and AI Services
![text](./images/mobile/ml-1.png)
- Can use `Rekognition` to identify objects, perform facial analysis
- Can be used in an event driven architecture
- can add image andd video analysis to your application
- identify objects, people, text, scenes and activities in images and videos
- Process videos stored in Amazon `S3`
- Publish completion status to `Amazon SNS topic`
![text](./images/mobile/ml-2.png)
- `Amazon transcribe` - add speech to text capabilities to applications
    - recorded speech can be converted to text before it can eb used 
    - Uses a deep learning process call Automatic speech recognition to convert speech to text quickly and accurately
![text](./images/mobile/ml-3.png)
- `Amazon translate` - neural machine translation service that delivers fast, high quality and affordable language translation
    - uses deep learning models to deliver more accurate adn more natural sounding translation
    - localize content such as website and application for your diverse users
- `Amazon Comprehend` - NLP
    - uses machine learning to uncover information in unstructured data
    - can identify critical elements in data, including references to language people and places and text files can be categorized in relevant topics
    - in real time, you can automatically and accurately detect customer sentiment in your content
![text](./images/mobile/ml-4.png)
- `Amazon lex` - conversational AI for chatbots
    - build conversational interfaces into any application using voice and text
    - build bots to increase contact center productivity, automate simple tasks and drive operational efficiencies       
- `Amazon DevOps Guru`
    - cloud operations service for improve application operation performance and availability
    - detect behaviors that deviate from normal operation pattern
    - Benefits:
        - Automatically detect operational issues
        - Resolve issue with ML-powered insights
        - Elastically scale operation analytics
        - Uses ML to reduce alarm noise
![text](./images/mobile/ml-5.png)
- `Amazon CodeGuru Security`
    - detect, track and fix code security vulnerabilities anywhere in the development cycle using ML and automated reasoning
    - Integrates with IDEs and CI/CD tools
    - Automated bug tracing
    - Assisted remediation through suggested code fixes
    - Offers performance optimization recommendations
    - Detects anomalies in application profiles

### Process and Analyze Videos
![text](./images/mobile/images.png)
1. user uploads an image
2. triggers `Lambda` which calls `Rekognition`
3. `Rekognition` analyzes the image
4. `Lambda` receives the answer and send the results to `DynamoDB`

### AWS License Manager
![text](./images/mobile/liscence-manager.png)
- Used to manage licenses from software vendors
    - Microsoft, oracle etc...
- Centralize management of software licenses for AWS and on prem
- Can track license usage including when licensed based on vCPUs, sockets or number of machines
- Distribute activate and track software licenses across accounts and throughout organization
- Enforce limits across multiple regions

### AWS Compute Optimizer
![text](./images/mobile/optimizer-1.png)
- Recommends optimal AWS resources for your workloads to reduce cost and improve performance
- Uses ML to analyze historical utilization metrics
- Offer optimization guidance for:
    - `EC2`
    - `EBS`
    - `Lambda`
- Results can be viewed in console or CLI
![text](./images/mobile/optimizer-2.png)

### AWS Budgets
- Allows you to set a custom budget and get notified when likely to reach or exceed the custom budget
    - can get an email or text etc...
    - integrated with `Cost Explorer`, `Q/Chatbot` and `Service Catalog`

### AWS Cost Explorer
- Allows you to see a breakdown of what has been happening in the AWS account
    - breaks down by service and by month, day, year etc...
    - can filter by `Region`, instance type etc...
    - supports cost categories and cost allocation tags
    - can see who created the costs etc...


### Cost Allocation Tags
- tag that can be used to organize resources and identify the cost
- can itemize the reports by the tag

### AWS Cost Management Tools
![text](./images/mobile/management-tool-1.png)
- `Costs Explorer` is a fre to that allows you to view and chart your costs
- Cna view cost data for past 13 months and forecast how much you are likely to spend over the next 3 months
- Can be used to discover patterns in how much you spend on AWS resources over time and to identify cost problem areas
- `Cost explorer` can help you to identify service usage statistics such as:
    - services used the most 
    - metrics for which AZ has the most traffic
    - which linked account is used the most

- `AWS Cost and Usage Report (CUR)`
    - publish AWS billing reports to `S3` bucket
    - reports break down costs by: hour, day ,month, product, product resource, tag
    - can update the report 3 times per dat
    - Create, retrieve and delete your reports using AWS `CUR` API reference

![text](./images/mobile/management-tool-2.png)
- AWS price list API
    - query the prices of services
    - Alerts va `Amazon SNS` when the price changes
