# VPC - Networking

## What is VPC?

A VPC (Virtual Private Cloud) is our own private network inside AWS. We decide the IP range, subnets and routing, and our resources like EC2 and RDS run inside it. Every region comes with a default VPC, but for real projects we usually create our own.

## CIDR

CIDR is how we write the IP range of the VPC and subnets. The number after the `/` tells how many bits are fixed, so a smaller number means more IPs.

- VPC: `10.0.0.0/16` gives 65,536 IPs
- Subnet: `10.0.1.0/24` gives 256 IPs (AWS reserves 5 in each subnet, so 251 usable)

A VPC can be between /16 and /28 in size.

## Subnets

A subnet is a smaller range inside the VPC and it lives in one Availability Zone. Example split of `10.0.0.0/16` in ap-south-1:

| Subnet | CIDR | AZ |
|--------|------|----|
| public-1 | 10.0.1.0/24 | ap-south-1a |
| public-2 | 10.0.2.0/24 | ap-south-1b |
| private-1 | 10.0.11.0/24 | ap-south-1a |
| private-2 | 10.0.12.0/24 | ap-south-1b |

```bash
aws ec2 create-subnet --vpc-id vpc-0abc123 --cidr-block 10.0.1.0/24 --availability-zone ap-south-1a
```

## Route tables

A route table decides where traffic goes. Each subnet is linked to one route table. Every table has a `local` route so subnets in the VPC can talk to each other. A public route table also has `0.0.0.0/0 -> igw-xxxx`.

## Internet Gateway

An Internet Gateway (IGW) connects the VPC to the internet. One IGW per VPC. It allows traffic both ways for instances that have a public IP and a route to the IGW.

## NAT Gateway

A NAT Gateway lets instances in a private subnet go out to the internet (for example to run `yum update` or pull Docker images) but nobody from outside can start a connection to them. It sits in a public subnet and the private route table has `0.0.0.0/0 -> nat-xxxx`. It costs money per hour, so delete it after practice.

## Security Groups

Security groups work at the instance (ENI) level. They are stateful and only support allow rules. We can also reference another security group as source, like "allow port 3306 only from the app SG".

## Network ACLs

NACLs work at the subnet level. They are stateless, so return traffic must be allowed separately (ephemeral ports 1024-65535). Rules are checked in number order and they support both allow and deny.

| | Security Group | Network ACL |
|---|---|---|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and deny |
| Order | All rules checked | Lowest number first |

## Public vs private subnet

| | Public subnet | Private subnet |
|---|---|---|
| Route to internet | 0.0.0.0/0 via IGW | 0.0.0.0/0 via NAT (or none) |
| Reachable from internet | Yes, if public IP and SG allow | No |
| Usually holds | Load balancers, bastion, NAT GW | App servers, databases |
