# AWS Notes

## S3
- Buckets
  - Bucket names have to be unique across the whole of S3 - globally unique
  - Versioning and logging can be enabled on all buckets
  - Once versioning is enabled, it can only be suspended and not disabled
  - Tags can be added to all buckets, 10 max
  - Encryption at rest (storage security) can be enabled for S3 buckets
    - AWS managed keys are taken care of by AWS
    - KMS managed keys are managed by the client
  - Buckets can host static website files with DNS used to present a friendly URL
  - Buckets can have a JSON based policy applied to them
  - Folders are basically prefixes to the name and are syntactic sugar rather than actual folders
    - Folders that are created are stored as objects so that you can navigate to them but they are just prefixes
- S3 objects
  - The minimum size of an object in S3 is zero bytes
  - Lifecycle rules can be applied to S3 objects, limited by tags or prefixes if desired
    - transition to infrequent-access or glacier after a set number of days
  - S3 objects can be deleted upon which they are marked for deletion.  The can be recovered once deleted
  - Cold objects can be migrated to Glacier automatically
  - A Tape Gateway, part of the Storage Gateway, can be used to backup data to S3 and Glacier using existing tape processes
- S3 Enhanced Features include
  - Intelligent tiering implemented through Lifecycle rules.  This allows for automated transition to tiered storage and eventual expiration if needed
  - Object locking to implement WORM (Write Once Read Many).  This is set up on the bucket when created and is permanent once enabled. When an object is uploaded into this bucket it can then be locked
  - Batch operations are for batch management of the buckets with automated actions

## Amazon Glacier
- Archival of data that is not needed on a regular basis for pennies per GB per month
- There are three access methods
  - Expedited (3-5 minutes)
  - Standard (3-5 hours)
  - Bulk (5-12 hours)
- S3 cold data can be moved into Glacier automatically
- Snow devices can be used to import data into Glacier
- Storage Gateways can be used to connect to Glacier for both access and archiving
- Archives are the things that you store in Glacier.  They are known as objects in S3 but archives in Glacier
- Vaults
  - this is where the archives are stored and are referenced through an ARN (Amazon Resource Name)
  - Vault Locks are for both securing the data and ensuring that Glacier is not used for regular data retrieval which can be expensive
  - Event Notifications, through SNS, can be requested so that you know when someone accesses the Vault
- Up to 5% can be retrieved from Glacier each month without cost
- Retrieval limit rates can be set if required
- A single AWS account can create up to 1000 Vaults per region
- Only empty Vaults can be deleted
- Multi-part upload is supported

## Elastic Block Store (EBS)
- Used for durable storage in EC2 instances
- Cannot be shared across instances apart from volume recovery
- Block level storage from one AWS service to another
- Volume Types include
  - Magnetic which were the original volume type.  They are the slowest and cheapest
    - Standard
    - Throughput optimised which is faster than standard
    - Cold which is very large and very slow
  - SSD General purpose
  - SSD Provisioned IOPS (PIOPS) which gives you selectable IOPS from 100 to 32000
- An EBS optimised EC2 instance should be used to take full advantage of an SSD
- Snapshots can be used to take backups and recover from that backup as well as provision new volumes for new EC2 instances
- Volume recovery means that a volume can be connected to another EC2 instance as a secondary volume

## Elastic File System (EFS)
- EFS is shareable and multiple EC2 instances can access a EFS share using NFSv4
- EFS is not supported on Windows instances, only Linux instances can access them
- EFS File Systems are normally launched into a VPC and can be mounted into separate AZs
- Throughput can be set to Bursting or Provisioned
- The EFS can be encrypted if required
- On Linux EC2 instances the EFS is mounted as an NFS mount
- Through the VPC an endpoint, powered by PrivateLink, can be created to share EFS through an ENI
  - this will set up a chargeable public IP address for the endpoint to which clients can connect

## Amazon FSx
- This is a simple and fully managed file system for the cloud
- You can choose between a Windows File Server and a Lustre high-performance file system
- It implements SMB so has native compatibility with Windows services
- Minimum size is 32GB and maximum size is 65536GB

## AWS Storage Gateway
- a software appliance (VM) that provides three types of storage solutions
  - file gateway to put files into S3
  - volume gateway using iSCSI to access storage volumes in AWS
    - cached stores the volume in AWS with a cache in the storage gateway
    - stored will store the volume in the storage gateway and save periodic backups to AWS
  - tape gateway 
- The storage gateway can be installed on-premise to give your private network access to AWS storage
- The storage gateway can be installed on an EC2 instance to share storage across VPCs

## Storage Comparison
- Per Operation Latency
  - EFS : low and consistent
  - S3  : low for mixed request types and for integration with CloudFront
  - EBS : lowest and consistent
- Throughput scale (how fast can the throughput scale up)
  - EFS : multiple GBs per second
  - S3  : multiple GBs per second
  - EBS : single GBs per second but note that we do not normally access very large files through EBS
- Data Availability and Durability
  - EFS : stored redundantly across multiple AZs
  - S3  : stored redundantly across multiple AZs
  - EBS : stored redundantly in a single AZ
- Access
  - EFS : one to thousands of EC2 instances from multiple AZs concurrently
  - S3  : one to millions of connections over the web
  - EBS : single EC2 instance in a single AZ
- Use Cases
  - EFS : will work for most use cases for shared drives
  - S3  : content, media, backups and data lakes
  - EBS : boot volumes, databases, data warehouses and ETL

## AMIs (Amazon Machine Images)
- the term instance indicates the use of an AMI
- generatd AMIs are
  - implicit (default) which means that only the owner can use them
  - explicit means that named users can use it
  - public means that anyone can use it
- virutalisation types are 
  - HVM hardware virtual machines where the virtualisation is supported in hardware VT-x (Intel) or AMD-V (AMD)
  - para virtual machines are not hardware assisted
- instance root volume contains the boot sector
  - instance store backed AMI means that the root volume is stored in S3 buckets
    - no support for the stop action, you're always terminating
    - data in the instance store is lost on failure
    - EBS volumes are preserved though
  - EBS backed AMI means that the root volume is stored in an EBS volume
    - supports the stop action
    - data is not lost during failure

## EC2 Instance Types
- General Purpose are T2, M3, M4, M5
  - T2 provides burst capability through credits accrued during quiet periods
  - M3, M4, M5 do not have any burst capability
- Compute Optimised are C3/4/5
- Memory Optimised are X1e, X1, R4 and R3
- Storage Optimised H1, I3 and D2
  - note that you can also choose memory optimised and implement the storage type that you need
- Advanced Computing are P3, P2, G3 and F1
  - these include GPUs and FPGAs
- Dedicated Instances
  - an EC2 instance that runs on hardware that is dedicated to a single customer
  - may share hardware with other EC2 instances from the same AWS account that are not dedicated
- Dedicated Hosts
  - used to launch EC2 instances on physical servers that are dedicated for your use
  - allow you to use existing software licences that are on that physical server

## EC2 Pricing Categories
- On Demand : available immediately and charged per minute of use.  Not much of a discount for using these
- Reserved  : pay up front for a reserved instance for a period of 1 month to 36 months. Large discounts available
  - includes no, partial and full up front costs which are most expensive to cheapest respectively
- Spot      : available at a reduced rate if there is spare capacity in EC2. May be shutdown if another person is willing to pay more
- Dedicated : not multi-tenanted.  Dedicated to you
- Scheduled : reserve instances on a recurring schedule for compute capacity when you know you will need it
- Capacity Reservation : reserve capacity for a highly critical workload that must be run at a particular time

## Security Groups
- security groups are stateful EC2 instance level firewalls
- they define protocols inbound from ip sources, such as 0.0.0.0/0
- supports allow rules only; deny rules are implicit so only allow rules need to be defined
- there can be up to five security groups for each ec2 instance
- security groups have outbound rules as well, but 'return' traffic is always permitted so the outbound rule may not be needed; i.e. they are stateful
- security groups to not apply to subnets
- all rules are evaluated before a decision is made to allow or deny traffic
- security groups must be applied to EC2 instances
- by default security groups are only bound to the primary network interface

## Network Access Control List (NACL)
- NACLs are stateless subnet level firewalls
- supports both allow and deny rules
- rule precedence applies and the first rule that matches is applied, either allow or deny
- rules are processed in order and processing stops as soon as a allow or deny rules matches
- the default rule number is *
- NACL rules apply to all EC2 instances inside a subnet

## SSH connection
- you must download the store the private key as a PEM and change its permissions to 400
- then connect using the following command:
```bash
ssh -i mykeypair.pem ec2-user@ip.address.of.instance
```

## Status Checks
- system status checks are for AWS to resolve
- instance status checks are os level issues for the user to resolve
- if any check fails then the status is set to impaired

## MetaData
- instance meta data is data about your instance that you can use to configure or manage running instances and is divided into categories
- *within* an instance it is available at http://169.254.169.254/latest/meta-data/ (trailing / is important)
- note that 169.254.169.0/24 is a link-local address and only available to a local host
- user meta data is data that is passed to the instance when created to run on boot (under advanced details)
- *within* an instance it is available at http://169.254.169.254/latest/user-data (trailing / is not needed)

## IP Addresses
- private ip addresses are not charged for and are used in both public and private subnets. Private IP addresses are retained when the instance is stopped. They are known to the OS
- public ip addresses are *dynamic* ip addresses. They are not charged for and are lost when the EC2 instance is stopped.  They are associated with a private ip address on the EC2 instance
- elastic ip addresses are *static* public ip addresses. You are charged if they are not used. They are associated with a private ip address on the EC2 instance or elastic network interfaces
- Public and elastic IP addresses are not known to the OS and are associated with the EC2 instance private IP outside the OS
  - the mapping to the private ip address from the public ip address is a NAT operation that is performed by the Internet Gateway 

## VPC
- A VPC (virtual private cloud) is a virtual network for your AWS account
- The IP range is separated from other VPCs in AWS
- A VPC can span multiple Availability Zones and has a number of subnets, public or private
- DirectConnect can be used to connect between VPCs or between on-premises network and a VPC with a VPN
- VPC peering can also be used to connect VPCs
- VPC endpoints can be created to allow access to resources controlled by policies
- Amazon recommends not deleting the default VPC that is created for you
- EIPs are Elastic IPs that are allocated from the Amazon Pool or can be provided by the customer themselves
  - they are charged for once allocated regardless of whether they are being used or not
- An ENI is a virtual network interface attached to an EC2 instance
  - multiple ENIs connected to an EC2 instance allow for dual homing
  - the ENI can then be given an IP address on any VPC subnet or public facing internet
- AWS Endpoints connect VPCs to AWS managed services
  - policies can be enforced on these endpoints
  - endpoint creation requires 
    - the source VPC and the target server in format com.amazonaws.region.service
    - policies for use of the endpoint (full access by default)
    - a route table

## VPC Peering
- this is the ability to connect two VPCs together so that instances can communicate directly
- the VPC peering must be requested by the VPC owner
- CIDR blocks cannot overlap in the VPCs that are to be peered
- new Security Groups may need to be set up or modified for the peering to work
- peering is not transitive, it is 1:1

## Subnet
- A subnet is a range of IP Addresses in your VPC
- A subnet must reside in a single Availability Zone
- Each subnet comes with a default routing table

## Public Subnet
- we attach an internet gateway to a public subnet
- we have a route table assigned to the public subnet that has a route that points to that internet gateway
- the ec2 instances have public ip addresses
- the subnet is defined to always assign public ip addresses to instances that are launched into it

## Route Table
- A set of rules that determine where IP traffic is sent
- Default routing tables are shared across subnets
- You can target destinations of 0.0.0.0/0 to the IGW to allow internet access from this route table

## Internet Gateway (IGW)
- A gateway attached to your VPC that enables communication betweeen resources in the VPC and the internet
- Performs NAT for those resources that have internet addresses

## VPC EndPoint
- Used to connect privately to your VPC without IGW, NAT or VPN connections

## ELB 
- Elastic Load Balancer that distributes incoming traffic to resources across multiple availability zones in a region
- Maintains a public IP so that your resources do not have to
- THere are three types of ELB, Classic Load Balancer, Network Load Balancer and Application Load Balancer
- Classic Load Balancer
  - this is the oldest type and is no longer recommended
- Application Load Balancer
  - best suited for HTTP and HTTPS loads
  - OSI Layer 7 
- Network Load Balancer
  - best suited for TCP/UDP/TLS where extreme performance is required
  - OSI Layer 4
- Load balacing algorithms include
  - round robin
  - randomized
  - centrally managed
  - threshold based
- Load balancing can be used with 
  - EC2
  - ECS
  - Auto Scaling
  - CloudWatch
  - Route 53

## NATs
- used to allow private instances access to the internet
- there are two - NAT instances (the old way) and NAT Gateways (the new way)
- NAT instances are multi-homed
  - NAT instances need an ENI and an EIP to connect to the internet
- NAT Gateways are paid for while in use and are recommended by AWS over NAT instances
  - NAT Gateways connect to a specific subnet
  - NAT Gateways will need an EIP
  - an update to the route table for the VPC is needed targeting 0.0.0.0/0

## Virtual Private Gateway (VPG)
- connects a local network to a VPC and is on the AWS side, inside the VPC
- the customer side needs a Customer Gateway which is either a physical device or a software application
- other ways to set up a VPN connection include
  - AWS hardware VPN
  - AWS Direct Connect
  - VPN CloudHub
  - Software VPN using protocols that AWS supports such as L2TP (Layer 2 Tunneling Protocol) with IPSec
  
## Identity and Access Management (IAM)
- Resources are the things on which actions can be taken, e.g. EC2 instances
- Principals are the things that can take actions, and include
  - Users
  - Groups
  - Roles
- Principals are also called identities
- IAM users are entities created in AWS
  - Can be a person or a service
  - They have credentials which consist of a username, password or up to two access keys
- Policies, based on JSON, define what can be done
  - Identity based policies are given to users
  - Group based policies are attached to groups and are preferred to identity based policies
  - Resource based policies are given to resources and state that only defined users (across accounts?) can take action on those resources
- Policies define the actions that can be taken
- A Role is an identity that is granted permissions, but not permanently assigned to a user
- AWS root user has unlimited access and is not recommended for everyday use
  - root user access is required for changin billing information or for closing your account
  - other admin access can be granted to a specific user by putting them in an admin group or adding admin permissions for them
  - for admin access permission boundaries can be set that limit the maximum access that an admin user will have
  - the login link for an IAM user will include the account id in the URL and must be used to login e.g. https://123456789012.signin.amazon.com/console
  - access keys can be created for this account, SSH public keys can be uploaded and HTTPS git credentials for AWS CodeCommit can be created as well
- Policy processing
  - be default all requests are denied
  - explicit allow overrides the default
  - permission boundaries can override explicit allows
  - explicit denies override explicit allows, for example a deny in one group will override an allow in another group
- Permission boundaries will limit the scope of any permissions but do not define specific permissions
- The principle of least privilege means that the minumum amount of permission should be given to users
- Role creation is used to give services such as EC2, CLIs and programs access to other services or accounts
- CloudTrail can be used to log events and generate alerts based on those events
  - by default the event history is stored for 90 days and then deleted unless you create a trail

## Auto Scaling Groups
- ASGs can span multiple AZs
- ASGs will first terminate those instances that have the oldest configuration or those that are about to hit the next billing hour
- There are four basic methods that can be used to launch auto scaling groups
  - using a Launch Template
    - the launch template includes all the parameters needed to launch EC2 instances
    - the IAM role using the launch template must have the ec2:RunInstances policy
  - using a Launch Configuration for ASGs
  - using an EC2 instance
    - effectively create an AMI for an EC2 instance
  - using the Amazon EC2 launch wizard

## DNS
- DNS Record types
  - A records are for IPv4
  - AAAA records are for IPv6
  - NS records resolve the name server for a given domain
  - MX records resolve the mail server for a given domain
  - CNAME records are aliases for the canonical names

## Route 53
- DNS management 
  - hosted zones can be public or private
    - private zones allow you to create a a domain for your VPC only
  Routing policies
    - simple is a direct translation of the domain name
    - weighted deals with multiple IP addresses and directs traffic to those based on user defined weights (e.g. 80% to a certain IP and 20% to the rest)
    - latency will pick the IP address with the lowest latency for that user
    - failover means use one and then failover if the first instance fails
    - geolocation directs users to the nearest host that will perform the best
    - nultivalue answer sets up a number of health checks and picks the one that passes the most

## Flow Logs
- also known as netflows
- can be written to an S3 bucket
- useful for monitoring and intrusion detection systems

## Kinesis
- Kinesis isued for the processing of data streams
- It provides realtime analytics and can enable the use of micro service applications
- Kinesis modes include
  - Kinesis Data Streams
    - ingests and *stores* data streams for processing by Kinesis Data Analytics, EMR, EC2 etc
  - Kinesis Data Firehouse
    - prepare and *stream* data continuously straight to whatever can process it
  - Kinesis Data Analytics
    - used to analyse realtime data streams
  - Kinesis Video Streams

## CloudFront
- provides a CDN for content acceleration and customisation using edge caches and edge servers
- can serve html assets from S3
- geo-restrictions or fencing can be implemented
- AWS Global accelerator works with TCP and UDP traffic with static IP addresses

## Web Application Firewall
- this is a component that can be configured to control access to HTTP or HTTPS requests
- it can work with CloudFront or a public ELB
- it can be deny by default or allow by default
- it will reply with a HTTP 403 or other configurable default behaviour
- Shield is a new subset that has some free tier options to protect against DDoS attacks

## Simple Queue Service (SQS)
- used to decouple applications via asynchrous messaging 
- they can include up to 256Kb of data
- queue types include
  - standard that does not guarantee message order
  - FIFO that does guarantee order but at a lower rate

## Simple Notification Service (SNS)
- used the pub sub model to send messages to clients
- publishers push messages to topics which are then broadcast
- subscribers receive messages when they subscribe to topics
- SNS stores across multiple AZs
- message delivery can be through HTTP/S, email, SMS, Lambda and SNS
- messages are limited to 256Kb or 140 bytes over SMS up to a maximum of 1600 bytes

## Simple Workflow (SWF)
- defines the sequence of events required to achieve an outcome
- used in decoupled applications
- operates in a logical domain to constrain its activities
- soon to be replaced by Step Functions

## Step Functions
- this is recommended over SWF
- they use state machines
- they can include nested Step Functions

## OpsWorks
- define configuration management 
- Configure
  - instance deployment
  - service deployment
  - application deployment
- Operate
  - application updates
  - infrastructure updates
- OpsWorks includes the following
  - OpsWork stacks
    - initial version that consists of a series of layers that can be combined into a stack
  - OpsWork Chef Automate
    - cookbooks with recipes
  - OpsWork Puppet
    - deployment of a master server with modules

## Cognito
- single sign-on and SSO
- Google, FB, Amazon and AD/SAML auth services
- profile management

## Elastic Map Reduce (EMR)
- Map reduce for job processing
- Master node will distribute to Core or Task Nodes
  - Core nodes include processing and storage capabilities
  - Task nodes include processing capabilities only

## CloudFormation
- CloudFormation can deploy entire solutions automatically
- CloudFormer is a drag and drop deployment creation tool
- New properties get added over time

## CloudWatch
- monitors cloud and on-premises solutions
- based on the monitoring of logs
- you can create a dashboard for your needs
- can generate an event and trigger alarms if needed

## TrustedAdvisor
- dashboard for alerting issues with resources and accounts in areas of
  - cost optimisation
  - performance
  - security
  - fault tolerance
  - service limits

## AWS Organisations
- allows an organisation to manage multiple accounts
- can add policies at the OU level rather than within the role level

## AWS Relational Databse Services (RDS)
- managed databases instances
- scaling is done through
  - read replicas
  - manual scaling through scripts