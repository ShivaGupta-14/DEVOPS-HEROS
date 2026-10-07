# EC2 - Compute

## What is EC2?

EC2 (Elastic Compute Cloud) gives us virtual servers in the cloud. We can start one in a couple of minutes, pick the size we want and pay only for the time it runs. It is basically renting a computer from AWS.

## AMI

An AMI (Amazon Machine Image) is the template used to launch an instance. It has the OS and any pre-installed software. Examples are Amazon Linux 2023, Ubuntu, or a custom AMI we build ourselves. AMI IDs are different in each region.

```bash
aws ec2 describe-images --owners amazon --filters "Name=name,Values=al2023-ami-*" --region ap-south-1
```

## Instance types

The instance type decides CPU, memory and network. The name tells you a lot, for example `t3.micro` means family `t`, generation `3`, size `micro`.

| Family | Good for | Example |
|--------|----------|---------|
| t | General, burstable, small apps | t3.micro |
| m | General purpose | m7i.large |
| c | Compute heavy | c7g.xlarge |
| r | Memory heavy | r7i.large |
| g / p | GPU workloads | g5.xlarge |

Types ending with `g` (like c7g) use AWS Graviton (ARM) chips.

## Key pairs

A key pair is used to SSH into Linux instances. AWS keeps the public key and we download the private key (`.pem`) only once, so don't lose it.

```bash
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text --region ap-south-1 > my-key.pem
chmod 400 my-key.pem
ssh -i my-key.pem ec2-user@<public-ip>
```

## Security Groups

A security group is a virtual firewall for the instance. It is stateful, so if inbound traffic is allowed the reply goes out automatically. We only write allow rules, for example port 22 from my IP and port 80 from anywhere.

## EBS

EBS (Elastic Block Store) is the disk attached to the instance. It lives in one Availability Zone and stays even if the instance is stopped. Common volume type is `gp3`. We can take snapshots of EBS volumes for backup.

## Public vs private IP

- Private IP: used inside the VPC, every instance gets one, it stays the same for the life of the instance
- Public IP: used to reach the instance from the internet, it changes when you stop and start
- Elastic IP: a static public IP you can attach, useful when the IP must not change

## Instance lifecycle

`pending` -> `running` -> `stopping` -> `stopped` -> `terminated`

- Stop: instance is shut down, EBS data stays, no compute charge (EBS still charged)
- Reboot: same host, same IPs
- Terminate: instance is deleted, root EBS is deleted by default

```bash
aws ec2 stop-instances --instance-ids i-0abc123 --region ap-south-1
```

## Common use cases

- Hosting web servers and backend APIs
- Jenkins servers or self-hosted CI runners
- Bastion host to reach private servers
- Running Kubernetes worker nodes (EKS node groups use EC2)
