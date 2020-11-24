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
- there can be multiple security groups for each ec2 instance
- security groups have outbound rules as well, but 'return' traffic is always permitted so the outbound rule may not be needed; i.e. they are stateful
- security groups to not apply to subnets
- deny rules are implicit so only allow rules need to be defined

## Network Access Control List (NACL)
- NACLs are stateless subnet level firewalls
- supports both allow and deny rules
- rule precedence applies and the first rule that matches is applied, either allow or deny

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
- Application ELB
  - best suited for HTTP and HTTPS loads
- Network Load Balancer
- best suited for TCP/UDP/TLS where extreme performance is required

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
  