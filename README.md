# AWS Certified Solutions Architect Associate SAA-C03 Course Code
*By [Digital Cloud Training](https://digitalcloud.training/) - Course Author Neal Davis*

## IAM ###
- https://digitalcloud.training/aws-iam/
- Users: IAM users are individuals or applications that you create in AWS to grant them access to your AWS resources.
    - long term
- Groups: IAM groups are collections of IAM users that you can use to simplify permissions management by assigning permissions to the group instead of individual users.
- Roles: IAM roles are sets of permissions that you can assign to AWS resources (like EC2 instances) or federated users, allowing them to assume specific permissions without having long-term credentials.
    - short term
- Policies: IAM policies are documents that define permissions for users, groups, or roles, specifying which actions they can perform on which resources.
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
- Access Keys - programmatic 
    - used for CLI or other API endpoints
    - Access Key ID and Secret Key

### Permission Boundaries ###  
- Sets maximum permissions an entity can have
- Assigned to users and roles
- Even if you have a policy, the boundary will restrict the max permissions the user can have
- Helps to mitigate privilege escalation
  - ie: something who only has IAM flags could create a user and given them admin access
    - can give someone IAM Full access but add a permission boundary that says the users this user creates can only have same or lower permissions as the current user

### Evaluation Logic ###
 ![text](./images/iam/permission-logic.png)
1. explicitly deny
2. organization policy
3. resource policy
4. identity policy
5. IAm permission boundary
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
- IAM Policies are are written in JSON
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
    - Assigned automatically when in a public subnet
    - IP is lost when instance is stopped
    - Associate with a private instance address
- Call instances have private IP address
- Can assign an elastic Ip, which is a static public IP
    - you are charged while it is not used

### Subnet Types ###
- AZ can hold subnets, private and public
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

### Launching EC2 ### ###
- chose [instance type](https://aws.amazon.com/ec2/instance-types/)
- chose AMI (includes OS and software)
    - back by a snapshot (point in time backup)
    - create create our own custom AMI

### Instance Metadata ### ###
- can be curled by `curl http://[ip]/latest/meta-data
- includes various data such as AMI-ID, hostname, etc...
- 2 versions of the instance metadata service
    - v1 - older and less secure
    - v2 - default, newer, more secure, requires authorization via session token
- User Data - ability to supply a script to run when the VM launches
- can be configured in the management console or cli
- in the cli the user data is passed in via file
- must be base64 encoded
- limited to 16kb
- only runs the first time the instance is launched
- In AWS EC2, 169.254.169.254 is the default gateway for instances. It's a virtual IP address that allows instances to access the Instance Metadata Service (IMDS).
    - ex: `curl http://169.254.169.254/latest/meta-data/instance-id`
        - will give us the instance id
    - see [user-metadata-md](./aws-saa-code-main/amazon-ec2/user-data-metadata.md) for more

### User Data ###
- can be given to ec2 as a script to run on startup in order to configure the instance
- always converted to base64
- only runs one time when the instance starts
- max 16kb


### Access Keys vs Roles With EC2 ###
- AWS CLI uses access keys
    - Access Keys are associated with a user account and inherit that users permissions
    - **This can give ec2 instance permissions but access keys are long term credentials and stored in plain text**
    - Access keys are configured with `aws configure`
- Should instead use IAM roles for services
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

### EC2 Placement Groups ###
- A way to control how AWS deploys Ec2s and what AZ they are in
![text](./images/ec2/cluster-pg.png)
- `Cluster` - packs instances closely within the same AZ, as opposed to randomly
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
- when launching an ec2, the EC2 sits in the AZ and it attaches to an Adapter in the subnet
- can connect multiple Adapters from different subnets to the same ec2 but the subnets must be in the same AZ
- Cannot remap adaptors across AZ
![text](./images/ec2/network-interface-types.png)

### Types of IPs ###
- Public - dynamic, changes with stops and starts of instances
- Private - used in private subnet only
- Elastic (EIP) - a public ip that is static
    - can be assigned to network interfaces
    - can be remapped across AZ's
    - you get charged when these are allocated to the account but not in use

###  NAT For Public Addresses ###
- Network Address Translation (NAT)
- Public IP addresses do not exist anywhere on the EC2
- The public address is translated to the private and vice versa by the `Internet Gateway`
![text](./images/ec2/nat.png)

### Private Subnets and Bastion Hosts ###
- https://digitalcloud.training/ssh-into-ec2-in-private-subnet/
-  A `bastion host` in AWS is a special-purpose instance that acts as a secure gateway to your private network within a Virtual Private Cloud (VPC). It sits in a public subnet, meaning it has a public IP address and can be accessed from the internet.
- Within a region we have a vpc, az and public subnet
- the public subnet has a route table in it which has a route to the local network and route to public internet 
- private subnets cannot be connected to the internet, there is no route option
    - this is the default for new subnets

![text](./images/ec2/bastion-host.png)
- can launch 2 ec2s, 1 in public subnet and 1 in private subnet
    - have an internet gateway and want to communicate from the outside to a computer in the private subnet
    - can send a request to the ec2 in the public subnet which can then route the request from internet, across the private ip routes 

### NAT Gateways and Instance Overview ###
- Can be used with bastion host for outbound only connection
    - ie download software or outside service
    - no one from the outside should be able to connect
- We can deploy the `NATGateway` into the public subnet so that it can get it a public ip
    - this will allow it to proxy traffic from the private EC2 to the internet
    - **alway deploy nat gateways or nat instances into a public subnet**
    - the `NatGateway` ID is added to teh private subnet route table 
    - Managed service
    - Can scale horizontally
    - Has high availability, can be deployed in multi-az
    - outbound access only, no security groups
    - **needs elastic ip**
    - need to add a route to the route table in private route table

![text](./images/ec2/nat-gatway-for-bastion-host.png)
- `NAT Instance` - an ec2 that does the same thing as a `NatGateway` but runs from and `EC2` instance
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
    - EC2 does not charge for stopped instance but EBS volumes do charge
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
- Nitro is the underlying platform for next generation of EC2
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
- EBS volumes are per second with a min of 1 minute
- Reserved instances
    - term is 1 or 3 years
    - standard and convertible RI
        - standard: change AZ, instance size and network type with `ModifyReservedInstances` API
        - Convertible - can change all of standard and family, OS, tenancy, payment option with `ExchangeReservedInstances` API
    - can pay all up front, partial up front, no up front
    - can reserve capacity in a specific az but does not reserver capacity when specified by region
    - tenancy can be default or dedicated 
- On Demand Capacity reservation 
    - reserve in specific AZ, for any duration
    - mitigates risk of being unable to get On-Demand Capacity
    - Does not require any term commitments and can be cancelled at any time 
- Savings Plans 
    - Compute Savings Plan:  1 or 3 year, hourly commitment to usage of Fargate, Lamdba and EC2, any region, family, size, tenancy and OS
    - Ec2 Savings Plan:  or 3 year, hourly commitment to usage of EC2, within a selected region and instance family; any size tenancy and OS
- Spot Instances 
![text](./images/ec2/spot-instances.png)
- can ask for a spot block to get 1-6 hours of uninterrupted compute
- pricing is 30% to 45% discount 

### EC2 Pricing Use Cases ###
![text](./images/ec2/prcing-use-cases.png)

## Elastic Load Balancing and Auto Scaling ##
