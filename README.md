# Terraform_AWS
Provisioning AWS instance with Terraform script on 2 regions


# Terraform AWS Infrastructure Project Documentation

## Table of Contents
1. Project Overview
2. Prerequisites and Setup
3. AWS Configuration
4. Project Structure
5. Resource Configuration
6. Deployment Steps
7. Verification and Testing
8. References

## 1. Project Overview

This project demonstrates the creation of AWS infrastructure using Terraform, including EC2 instances across multiple regions, security groups, key pairs, and EBS volumes. The infrastructure is deployed to AWS account: ############.

### Project Goals
- Create EC2 instances in multiple AWS regions (us-east-1 and us-east-2)
- Configure security with proper key pairs and security groups
- Set up persistent storage with EBS
- Implement infrastructure as code best practices

## 2. Prerequisites and Setup

### Required Tools
- Terraform (v1.0 or later)
- AWS CLI
- Text editor (VS Code recommended)

### Installation Steps

#### Installing Terraform
1. Visit the official Terraform downloads page: https://developer.hashicorp.com/terraform/downloads
2. Download and install Terraform for your operating system
3. Verify installation:
```bash
terraform --version
```

#### Installing AWS CLI
1. Visit AWS CLI installation guide: https://aws.amazon.com/cli/
2. Follow installation instructions for your operating system
3. Verify installation:
```bash
aws --version
```

## 3. AWS Configuration

### Setting up AWS Credentials

1. Log into AWS Console: https://console.aws.amazon.com
2. Navigate to IAM → Users → Your User → Security Credentials
3. Create Access Key:
   - Click "Create access key"
   - Select "Command Line Interface (CLI)"
   - Acknowledge the security recommendations
   - Click "Next"
   - Add a description tag (optional)
   - Click "Create access key"
   - Save both the Access Key ID and Secret Access Key

4. Configure AWS CLI:
```bash
aws configure
```
Enter:
- AWS Access Key ID
- AWS Secret Access Key
- Default region: us-east-1
- Output format: json

To verify configuration:
```bash
aws sts get-caller-identity
```

## 4. Project Structure

### Initial Setup

1. Create project directory:
```bash
mkdir -p "/Users/dayosasanya/Desktop/Terraform /Project1"
cd "/Users/dayosasanya/Desktop/Terraform /Project1"
```

2. Create main configuration file:
```bash
touch main2.tf
```

### Files Created

1. `main2.tf` - Main Terraform configuration file
2. `terraform_key` - Private SSH key (generated)
3. `terraform_key.pub` - Public SSH key (generated)
4. `.terraform.lock.hcl` - Dependency lock file (auto-generated)
5. `terraform.tfstate` - State file (auto-generated)

### Directory Structure
```
Project1/
├── main2.tf
├── terraform_key
├── terraform_key.pub
├── .terraform/
├── .terraform.lock.hcl
└── terraform.tfstate
```

## 5. Resource Configuration

### Provider Configuration
Source: https://registry.terraform.io/providers/hashicorp/aws/latest/docs

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "ohio"
  region = "us-east-2"
}
```

### Security Group Configuration
Source: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group

```hcl
resource "aws_security_group" "web_security" {
  name        = "web_security_group"
  description = "Security group for web and SSH access"

  # SSH Access
  ingress {
    description = "SSH access"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTP Access
  ingress {
    description = "HTTP access"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTPS Access
  ingress {
    description = "HTTPS access"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Outbound Rules
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web_security_group"
  }
}
```

### Key Pair Creation
Source: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair

1. Generate SSH Key:
```bash
ssh-keygen -t rsa -b 2048 -f terraform_key -N ""
```

This command:
- Creates RSA key pair
- 2048-bit encryption
- Saves private key to 'terraform_key'
- Saves public key to 'terraform_key.pub'
- Empty passphrase for automation

2. Configure Key Pairs in Terraform:
```hcl
# Key Pair for us-east-1
resource "aws_key_pair" "terraform_key" {
  key_name   = "terraform-key-pair"
  public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlzW+KPttWgBUk7UMsKKE0yfDpFBkEy/bK15O6sGNt+Ec9kbH2LMRGm2M0E/GAWqea5Ry9UGkDaSU6EokZYXm64xTN0kkjfNxcm6GwKPcAyHzTMvBjBi0KjfueuKNRZmm4KAnEFk1rC1Wmp0NFZfWca8nhfTIKtIxKF3n23KozbzwSIgSh3zJFqHzuABQWkvj0n8kHPw3ciq2T4T3VriDtcM0n0aPOWyxIBEGJLPh/ONjsEn4E1cDaQNZpCoFSRR4Z8X4vhktRiX5uelUUIGqkY3JI+fLgGWhrngY9miO8YgOpF4HVW5k9g4dBOpEGvsdf3OWmE8rjrun9s+CAeF/L"

  tags = {
    Name = "terraform-key-pair"
  }
}

# Key Pair for us-east-2
resource "aws_key_pair" "terraform_key_ohio" {
  provider    = aws.ohio
  key_name    = "terraform-key-pair"
  public_key  = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlzW+KPttWgBUk7UMsKKE0yfDpFBkEy/bK15O6sGNt+Ec9kbH2LMRGm2M0E/GAWqea5Ry9UGkDaSU6EokZYXm64xTN0kkjfNxcm6GwKPcAyHzTMvBjBi0KjfueuKNRZmm4KAnEFk1rC1Wmp0NFZfWca8nhfTIKtIxKF3n23KozbzwSIgSh3zJFqHzuABQWkvj0n8kHPw3ciq2T4T3VriDtcM0n0aPOWyxIBEGJLPh/ONjsEn4E1cDaQNZpCoFSRR4Z8X4vhktRiX5uelUUIGqkY3JI+fLgGWhrngY9miO8YgOpF4HVW5k9g4dBOpEGvsdf3OWmE8rjrun9s+CAeF/L"

  tags = {
    Name = "terraform-key-pair"
  }
}
```

### EC2 Instance Configuration
Source: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance

```hcl
# EC2 instance in us-east-1
resource "aws_instance" "secondterraform" {
  ami           = "ami-0e731c8a588258d0d"  # Amazon Linux 2023 AMI
  instance_type = "t2.micro"
  key_name      = aws_key_pair.terraform_key.key_name

  vpc_security_group_ids = [aws_security_group.web_security.id]

  tags = {
    Name = "secondterraform"
  }
}

# EC2 instance in us-east-2
resource "aws_instance" "thirdterraform" {
  provider      = aws.ohio
  ami           = "ami-0cd3c7f72edd5b06d"  # Amazon Linux 2023 AMI
  instance_type = "t2.micro"
  key_name      = aws_key_pair.terraform_key_ohio.key_name

  tags = {
    Name = "Thirdterraform"
  }
}
```

### EBS Volume Configuration
Source: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ebs_volume

```hcl
# Create EBS Volume
resource "aws_ebs_volume" "extra_storage" {
  availability_zone = aws_instance.secondterraform.availability_zone
  size             = 20  # 20 GB
  type             = "gp3"  # General Purpose SSD

  tags = {
    Name = "ExtraStorage"
  }
}

# Attach EBS Volume to EC2
resource "aws_volume_attachment" "storage_attachment" {
  device_name = "/dev/xvdf"
  volume_id   = aws_ebs_volume.extra_storage.id
  instance_id = aws_instance.secondterraform.id
}
```

## 6. Deployment Steps

1. Initialize Terraform:
```bash
terraform init
```
This downloads required providers and initializes the working directory.

2. Review the execution plan:
```bash
terraform plan
```
This shows what changes Terraform will make to your infrastructure.

3. Apply the configuration:
```bash
terraform apply -auto-approve
```
This creates all the resources in AWS.

### Actual Resources Created:
- Security Group: sg-05d885021607ecbd2
- EC2 Instances:
  * us-east-1: i-04b906b68f82d6c71
  * us-east-2: i-039e3b115192134ed
- EBS Volume: vol-020297e616ea62237

## 7. Verification and Testing

### Verify EC2 Instances
1. Open AWS Console: https://console.aws.amazon.com/ec2/
2. Check instances in both regions:
   - us-east-1 (N. Virginia):
     * Name: "secondterraform"
     * Instance ID: i-04b906b68f82d6c71
   - us-east-2 (Ohio):
     * Name: "Thirdterraform"
     * Instance ID: i-039e3b115192134ed

### Verify Security Groups
1. Navigate to Security Groups
2. Find "web_security_group" (sg-05d885021607ecbd2)
3. Verify rules:
   - Inbound:
     * SSH (22)
     * HTTP (80)
     * HTTPS (443)
   - Outbound:
     * All traffic

### Verify EBS Volumes
1. Navigate to Volumes
2. Find volume with ID: vol-020297e616ea62237
3. Verify:
   - Size: 20 GiB
   - Type: gp3
   - Attached to: i-04b906b68f82d6c71

## 8. References

- Terraform AWS Provider Documentation:
  * Main: https://registry.terraform.io/providers/hashicorp/aws/latest/docs
  * EC2: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance
  * Security Groups: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group
  * Key Pairs: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair
  * EBS Volumes: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ebs_volume

- AWS Documentation:
  * EC2: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html
  * Security Groups: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html
  * EBS: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html

- Tools Documentation:
  * Terraform: https://developer.hashicorp.com/terraform/docs
  * AWS CLI: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html
