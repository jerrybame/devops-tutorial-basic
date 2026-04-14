# devops-tutorial-basic

# DevOps Training

This project contains Terraform configuration to provision two EC2 instances on AWS along with all the necessary networking and security resources.

## What Gets Created

### Networking
- **VPC** — a dedicated Virtual Private Cloud with CIDR `10.0.0.0/16`, isolating all resources in their own network
- **Internet Gateway** — attaches to the VPC to allow outbound internet access
- **Public Subnet** — a subnet (`10.0.1.0/24`) in availability zone `us-east-1a` where the EC2 instances live; instances get a public IP automatically
- **Route Table** — routes all outbound traffic (`0.0.0.0/0`) through the internet gateway and is associated with the public subnet

### Security
- **Security Group** — acts as a virtual firewall for the EC2 instances:
  - Allows inbound SSH (port 22) from a configurable CIDR
  - Allows inbound HTTP (port 80) from anywhere
  - Allows all outbound traffic

### Compute
- **Key Pair** — registers your local SSH public key with AWS so you can SSH into the instances
- **2 x EC2 Instances** — Amazon Linux 2 `t2.micro` instances, each with:
  - A 20 GB encrypted `gp3` root volume
  - Placed in the public subnet with a public IP
  - Protected by the security group above

## File Structure

```
terraform/
├── main.tf           # All resource definitions
├── variables.tf      # Input variable declarations and defaults
├── outputs.tf        # Values exposed after apply (IPs, IDs, DNS)
└── terraform.tfvars  # Your variable overrides
```

## Usage

```bash
cd devops-training/terraform

# Download required providers
terraform init

# Preview what will be created
terraform plan

# Create the resources
terraform apply

# Destroy everything when done
terraform destroy
```

## Configuration

Edit `terraform.tfvars` to customise the deployment:

| Variable          | Default         | Description                          |
|-------------------|-----------------|--------------------------------------|
| `aws_region`      | `us-east-1`     | AWS region to deploy into            |
| `project_name`    | `devops-training` | Prefix used for all resource names |
| `instance_type`   | `t2.micro`      | EC2 instance size                    |
| `ami_id`          | Amazon Linux 2  | AMI to use (region-specific)         |
| `public_key_path` | `~/.ssh/id_rsa.pub` | Path to your SSH public key      |
| `allowed_ssh_cidr`| `0.0.0.0/0`     | IP range allowed to SSH in           |

> **Security tip:** Set `allowed_ssh_cidr` to your own IP (`x.x.x.x/32`) before deploying to avoid exposing SSH to the internet.

## Outputs

After `terraform apply` completes, the following values are printed:

- `instance_ids` — AWS instance IDs for both EC2s
- `instance_public_ips` — public IP addresses to connect to
- `instance_public_dns` — public DNS hostnames
- `vpc_id` — ID of the created VPC
- `security_group_id` — ID of the security group

## SSH Access

```bash
ssh -i ~/.ssh/id_rsa ec2-user@<instance_public_ip>
```
