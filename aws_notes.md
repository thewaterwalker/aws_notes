# AWS Notes

## Instance Types
 - On Demand : available immediately and charged per minute of use.  Not much of a discount for using these
 - Reserved  : pay up front for a reserved instance for a period of 1 month to 36 months. Large discounts available
 - Spot      : available at a reduced rate if there is spare capacity in EC2. May be shutdown if another person is willing to pay more
 - Dedicated : not multi-tenanted.  Dedicated to you
 - Scheduled : reserve instances on a recurring schedule for compute capacity when you know you will need it
 - Capacity Reservation : reserve capacity for a highly critical workload that must be run at a particular time

## Security Groups
 - instance level firewalls
 - they define protocols inbound from ip sources, such as 0.0.0.0/0
 - there can be multiple security groups for each ec2 instance
 - security groups have outbound rules as well, but 'return' traffic is always permitted so the outbound rule may not be needed

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
 - A VPC can span multiple Availability Zones

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
 - NAT Gateways are paid for while in use and are recommended by AWS over NAT instances

