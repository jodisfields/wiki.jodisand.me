# Terraform Cheatsheet

A comprehensive guide to Terraform - Infrastructure as Code (IaC) tool for building, changing, and versioning infrastructure.

## Table of Contents

- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [CLI Commands](#cli-commands)
- [Configuration Syntax](#configuration-syntax)
- [Variables](#variables)
- [Outputs](#outputs)
- [State Management](#state-management)
- [Modules](#modules)
- [Providers](#providers)
- [Best Practices](#best-practices)

## Installation

### Install Terraform

```bash
# macOS (using Homebrew)
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux (Ubuntu/Debian)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Verify installation
terraform version
```

## Core Concepts

### Infrastructure as Code (IaC)

- **Declarative**: Define desired state, not steps
- **Version Control**: Track infrastructure changes
- **Reusability**: Share and reuse configurations
- **Automation**: Reproducible deployments

### Terraform Workflow

```
1. Write → Configuration files (.tf)
2. Init → Initialize working directory
3. Plan → Preview changes
4. Apply → Execute changes
5. Destroy → Remove infrastructure (optional)
```

### Key Components

- **Providers**: Plugins for interacting with APIs (AWS, Azure, GCP, etc.)
- **Resources**: Infrastructure objects (EC2 instances, S3 buckets, etc.)
- **Data Sources**: Query existing infrastructure
- **Variables**: Parameterize configurations
- **Outputs**: Export values
- **Modules**: Reusable configurations
- **State**: Track managed resources

## CLI Commands

### Basic Commands

```bash
# Initialize working directory (download providers)
terraform init

# Validate configuration syntax
terraform validate

# Format configuration files
terraform fmt
terraform fmt -recursive

# Show execution plan
terraform plan
terraform plan -out=tfplan

# Apply changes
terraform apply
terraform apply tfplan
terraform apply -auto-approve

# Destroy infrastructure
terraform destroy
terraform destroy -auto-approve
terraform destroy -target=resource_type.resource_name

# Show current state
terraform show
terraform show -json

# List resources in state
terraform state list

# Show specific resource in state
terraform state show resource_type.resource_name
```

### Workspace Commands

```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new dev
terraform workspace new prod

# Switch workspace
terraform workspace select dev

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete dev
```

### State Management

```bash
# Pull remote state
terraform state pull

# Push local state to remote
terraform state push

# Remove resource from state
terraform state rm resource_type.resource_name

# Move resource in state
terraform state mv source destination

# Import existing resource
terraform import resource_type.resource_name resource_id

# Replace resource (force recreation)
terraform apply -replace=resource_type.resource_name

# Refresh state
terraform refresh
```

### Output Commands

```bash
# Show all outputs
terraform output

# Show specific output
terraform output output_name

# Output in JSON format
terraform output -json

# Show raw output (no quotes)
terraform output -raw output_name
```

### Advanced Commands

```bash
# View dependency graph
terraform graph | dot -Tpng > graph.png

# Get provider documentation
terraform providers

# Lock provider versions
terraform providers lock

# Console (interactive)
terraform console

# Taint resource (mark for recreation)
terraform taint resource_type.resource_name

# Untaint resource
terraform untaint resource_type.resource_name
```

## Configuration Syntax

### Basic Resource

```hcl
# provider.tf
terraform {
  required_version = ">= 1.0"

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

# main.tf
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name        = "WebServer"
    Environment = "Production"
  }
}
```

### Data Sources

```hcl
# Query existing resources
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

### Resource Dependencies

```hcl
# Implicit dependency (reference)
resource "aws_eip" "ip" {
  vpc      = true
  instance = aws_instance.web.id
}

# Explicit dependency
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  depends_on = [aws_security_group.allow_http]
}
```

### Conditional Expressions

```hcl
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.environment == "production" ? "t2.large" : "t2.micro"
}
```

### Loops

```hcl
# count (for identical resources)
resource "aws_instance" "server" {
  count = 3

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "Server-${count.index + 1}"
  }
}

# for_each (for unique resources)
resource "aws_iam_user" "users" {
  for_each = toset(["alice", "bob", "charlie"])

  name = each.value
}

# for_each with map
variable "instances" {
  type = map(string)
  default = {
    web = "t2.micro"
    db  = "t2.small"
  }
}

resource "aws_instance" "servers" {
  for_each = var.instances

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value

  tags = {
    Name = each.key
  }
}
```

### Dynamic Blocks

```hcl
resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules

    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))

  default = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}
```

## Variables

### Variable Declaration

```hcl
# variables.tf
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}

variable "enable_monitoring" {
  description = "Enable detailed monitoring"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default = {
    Environment = "dev"
    Project     = "myapp"
  }
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "instance_config" {
  description = "Instance configuration"
  type = object({
    type = string
    ami  = string
  })
  default = {
    type = "t2.micro"
    ami  = "ami-0c55b159cbfafe1f0"
  }
}

# Validation
variable "environment" {
  description = "Environment name"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# Sensitive variable
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}
```

### Setting Variables

```bash
# Command line
terraform apply -var="region=us-west-2" -var="instance_count=3"

# Variable file (terraform.tfvars or *.auto.tfvars)
region         = "us-west-2"
instance_count = 3
tags = {
  Environment = "production"
  Project     = "webapp"
}

# Load specific var file
terraform apply -var-file="prod.tfvars"

# Environment variables
export TF_VAR_region="us-west-2"
export TF_VAR_instance_count=3
terraform apply
```

### Variable Precedence (lowest to highest)

1. Default values in variable definitions
2. Environment variables (TF_VAR_*)
3. terraform.tfvars file
4. *.auto.tfvars files (alphabetical order)
5. -var-file flags (in order specified)
6. -var flags (in order specified)

## Outputs

### Output Declaration

```hcl
# outputs.tf
output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP address"
  value       = aws_instance.web.public_ip
}

output "instance_private_ip" {
  description = "Private IP address"
  value       = aws_instance.web.private_ip
  sensitive   = true
}

# Output from multiple resources
output "server_ids" {
  description = "List of server IDs"
  value       = aws_instance.servers[*].id
}

output "server_ips" {
  description = "Map of server IPs"
  value = {
    for instance in aws_instance.servers :
    instance.tags.Name => instance.private_ip
  }
}
```

### Using Outputs

```bash
# View all outputs
terraform output

# View specific output
terraform output instance_id

# Use in scripts
INSTANCE_ID=$(terraform output -raw instance_id)
echo "Instance ID: $INSTANCE_ID"

# JSON format
terraform output -json > outputs.json
```

## State Management

### Local State

```hcl
# Default: terraform.tfstate in working directory
# .gitignore should include:
# *.tfstate
# *.tfstate.backup
```

### Remote State (S3 Backend)

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

### Remote State Data Source

```hcl
# Reference another Terraform state
data "terraform_remote_state" "vpc" {
  backend = "s3"

  config = {
    bucket = "my-terraform-state-bucket"
    key    = "vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "web" {
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
}
```

### State Locking

```bash
# Force unlock (if lock is stuck)
terraform force-unlock <lock-id>
```

## Modules

### Module Structure

```
modules/
└── vpc/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

### Creating a Module

```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true

  tags = merge(
    var.tags,
    {
      Name = var.name
    }
  )
}

resource "aws_subnet" "public" {
  count = length(var.public_subnets)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnets[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = merge(
    var.tags,
    {
      Name = "${var.name}-public-${count.index + 1}"
      Type = "public"
    }
  )
}

# modules/vpc/variables.tf
variable "name" {
  description = "VPC name"
  type        = string
}

variable "cidr_block" {
  description = "VPC CIDR block"
  type        = string
}

variable "public_subnets" {
  description = "Public subnet CIDR blocks"
  type        = list(string)
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}

# modules/vpc/outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}
```

### Using a Module

```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"

  name               = "production-vpc"
  cidr_block         = "10.0.0.0/16"
  public_subnets     = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones = ["us-east-1a", "us-east-1b"]

  tags = {
    Environment = "production"
    ManagedBy   = "terraform"
  }
}

# Reference module outputs
resource "aws_instance" "web" {
  subnet_id = module.vpc.public_subnet_ids[0]
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

### Public Module Registry

```hcl
# Use module from Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
  enable_vpn_gateway = false
}
```

## Providers

### AWS Provider

```hcl
provider "aws" {
  region  = "us-east-1"
  profile = "default"

  default_tags {
    tags = {
      Environment = "production"
      ManagedBy   = "terraform"
    }
  }
}

# Multiple provider configurations
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

resource "aws_instance" "west_server" {
  provider = aws.west

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

### Azure Provider

```hcl
provider "azurerm" {
  features {}

  subscription_id = var.subscription_id
}

resource "azurerm_resource_group" "main" {
  name     = "my-resources"
  location = "East US"
}
```

### Google Cloud Provider

```hcl
provider "google" {
  project = var.project_id
  region  = "us-central1"
}

resource "google_compute_instance" "vm" {
  name         = "my-instance"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
}
```

## Best Practices

### Project Structure

```
project/
├── main.tf                 # Main configuration
├── variables.tf            # Input variables
├── outputs.tf             # Output values
├── provider.tf            # Provider configuration
├── backend.tf             # Backend configuration
├── terraform.tfvars       # Variable values (gitignored if sensitive)
├── versions.tf            # Version constraints
├── modules/               # Local modules
│   └── vpc/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── environments/          # Environment-specific configs
    ├── dev/
    │   └── terraform.tfvars
    ├── staging/
    │   └── terraform.tfvars
    └── prod/
        └── terraform.tfvars
```

### Version Constraints

```hcl
# versions.tf
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # >= 5.0, < 6.0
    }
    random = {
      source  = "hashicorp/random"
      version = ">= 3.0"
    }
  }
}
```

### Naming Conventions

```hcl
# Use descriptive, consistent names
resource "aws_instance" "web_server" {  # Good
  # ...
}

resource "aws_instance" "i1" {  # Bad
  # ...
}

# Use underscores for resource names
# Use hyphens for resource tags and names within cloud provider
resource "aws_s3_bucket" "app_logs" {
  bucket = "myapp-logs-prod"
}
```

### Security Best Practices

```hcl
# 1. Never commit sensitive data
variable "db_password" {
  type      = string
  sensitive = true
}

# 2. Use separate state per environment
# 3. Enable state encryption
terraform {
  backend "s3" {
    encrypt = true
  }
}

# 4. Use IAM roles instead of access keys
# 5. Implement least privilege access
# 6. Use .gitignore
*.tfstate
*.tfstate.backup
.terraform/
*.tfvars  # If contains sensitive data
override.tf
override.tf.json
```

### State Management Best Practices

1. **Use remote state** for team collaboration
2. **Enable state locking** to prevent conflicts
3. **Enable state encryption** for sensitive data
4. **Regular state backups**
5. **Separate state per environment** (dev, staging, prod)
6. **Never manually edit state** files

### Code Organization

```hcl
# Use locals for computed values
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
    Project     = var.project_name
  }

  name_prefix = "${var.project_name}-${var.environment}"
}

resource "aws_instance" "web" {
  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-web-server"
    }
  )
}
```

### Documentation

```hcl
# Add descriptions to variables
variable "instance_type" {
  description = "EC2 instance type for web servers"
  type        = string
  default     = "t2.micro"
}

# Add descriptions to outputs
output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = aws_lb.main.dns_name
}
```

### Testing

```bash
# Validate syntax
terraform validate

# Format code
terraform fmt -recursive

# Security scanning (using tfsec)
tfsec .

# Cost estimation (using Infracost)
infracost breakdown --path .

# Plan before apply
terraform plan -out=tfplan
terraform apply tfplan
```

## Common Patterns

### Multi-Environment Setup

```hcl
# Directory structure
environments/
├── dev/
│   ├── main.tf
│   ├── backend.tf
│   └── terraform.tfvars
├── staging/
│   ├── main.tf
│   ├── backend.tf
│   └── terraform.tfvars
└── prod/
    ├── main.tf
    ├── backend.tf
    └── terraform.tfvars

# Each environment uses the same modules
module "app" {
  source = "../../modules/app"

  environment = var.environment
  # ...
}
```

### Workspace-based Environments

```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Use workspace name in configuration
resource "aws_instance" "web" {
  count = terraform.workspace == "prod" ? 3 : 1

  tags = {
    Environment = terraform.workspace
  }
}
```

## Troubleshooting

### Common Issues

```bash
# Dependency errors
terraform apply -refresh=false

# Resource already exists
terraform import resource_type.name resource_id

# State locked
terraform force-unlock <lock-id>

# Provider plugin errors
rm -rf .terraform
terraform init

# Debug logging
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform.log
terraform apply
```

## Additional Resources

- [Official Terraform Documentation](https://www.terraform.io/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [Best Practices Guide](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)
- [Style Guide](https://www.terraform.io/docs/language/syntax/style.html)

---

*Last updated: 2025-11-16*
