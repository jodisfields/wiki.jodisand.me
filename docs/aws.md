# AWS (Amazon Web Services) Cheatsheet

A comprehensive guide to Amazon Web Services - the world's most comprehensive cloud platform.

## Table of Contents

- [AWS CLI](#aws-cli)
- [IAM](#iam)
- [EC2](#ec2)
- [S3](#s3)
- [VPC](#vpc)
- [RDS](#rds)
- [Lambda](#lambda)
- [DynamoDB](#dynamodb)
- [CloudFormation](#cloudformation)
- [ECS & EKS](#ecs--eks)
- [Common Patterns](#common-patterns)

## AWS CLI

### Installation & Configuration

```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# macOS
brew install awscli

# Verify installation
aws --version

# Configure AWS CLI
aws configure
# AWS Access Key ID: YOUR_ACCESS_KEY
# AWS Secret Access Key: YOUR_SECRET_KEY
# Default region name: us-east-1
# Default output format: json

# Configure named profile
aws configure --profile dev

# List configured profiles
aws configure list-profiles

# Use specific profile
aws s3 ls --profile dev

# Set default profile
export AWS_PROFILE=dev
```

### Common CLI Commands

```bash
# Get caller identity
aws sts get-caller-identity

# List regions
aws ec2 describe-regions --output table

# List availability zones
aws ec2 describe-availability-zones --region us-east-1

# Get account ID
aws sts get-caller-identity --query Account --output text

# Output formats
aws ec2 describe-instances --output json
aws ec2 describe-instances --output yaml
aws ec2 describe-instances --output table
aws ec2 describe-instances --output text

# Query with JMESPath
aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId'

# Filter results
aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro"
```

## IAM (Identity and Access Management)

### Users & Groups

```bash
# Create user
aws iam create-user --user-name john

# List users
aws iam list-users

# Create access key
aws iam create-access-key --user-name john

# Delete user
aws iam delete-user --user-name john

# Create group
aws iam create-group --group-name developers

# Add user to group
aws iam add-user-to-group --user-name john --group-name developers

# List groups for user
aws iam list-groups-for-user --user-name john

# Remove user from group
aws iam remove-user-from-group --user-name john --group-name developers
```

### Policies

```bash
# List policies
aws iam list-policies --scope Local

# Create policy
aws iam create-policy --policy-name MyPolicy --policy-document file://policy.json

# Attach policy to user
aws iam attach-user-policy --user-name john --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# Attach policy to group
aws iam attach-group-policy --group-name developers --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Detach policy from user
aws iam detach-user-policy --user-name john --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# Get policy
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# List attached user policies
aws iam list-attached-user-policies --user-name john
```

### Example IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-bucket"
    }
  ]
}
```

### Roles

```bash
# Create role
aws iam create-role --role-name MyRole --assume-role-policy-document file://trust-policy.json

# List roles
aws iam list-roles

# Attach policy to role
aws iam attach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Assume role
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/MyRole --role-session-name session1
```

## EC2 (Elastic Compute Cloud)

### Instance Management

```bash
# List instances
aws ec2 describe-instances

# List instances (specific fields)
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType,PublicIpAddress]' --output table

# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]'

# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Reboot instance
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Describe instance status
aws ec2 describe-instance-status --instance-ids i-1234567890abcdef0

# Get instance metadata (from within instance)
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

### AMIs (Amazon Machine Images)

```bash
# List AMIs
aws ec2 describe-images --owners self

# Create AMI from instance
aws ec2 create-image --instance-id i-1234567890abcdef0 --name "MyAMI"

# Deregister AMI
aws ec2 deregister-image --image-id ami-0123456789abcdef0

# Copy AMI to another region
aws ec2 copy-image --source-image-id ami-0123456789abcdef0 --source-region us-east-1 --region us-west-2 --name "MyAMI-Copy"
```

### Key Pairs

```bash
# Create key pair
aws ec2 create-key-pair --key-name my-key-pair --query 'KeyMaterial' --output text > my-key-pair.pem
chmod 400 my-key-pair.pem

# List key pairs
aws ec2 describe-key-pairs

# Delete key pair
aws ec2 delete-key-pair --key-name my-key-pair

# Import key pair
aws ec2 import-key-pair --key-name my-key-pair --public-key-material fileb://~/.ssh/id_rsa.pub
```

### Security Groups

```bash
# Create security group
aws ec2 create-security-group --group-name my-sg --description "My security group" --vpc-id vpc-0123456789abcdef0

# List security groups
aws ec2 describe-security-groups

# Add inbound rule (SSH)
aws ec2 authorize-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 22 --cidr 0.0.0.0/0

# Add inbound rule (HTTP)
aws ec2 authorize-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 80 --cidr 0.0.0.0/0

# Remove inbound rule
aws ec2 revoke-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 22 --cidr 0.0.0.0/0

# Delete security group
aws ec2 delete-security-group --group-id sg-0123456789abcdef0
```

## S3 (Simple Storage Service)

### Bucket Operations

```bash
# Create bucket
aws s3 mb s3://my-bucket

# List buckets
aws s3 ls

# List bucket contents
aws s3 ls s3://my-bucket
aws s3 ls s3://my-bucket/path/ --recursive

# Remove bucket
aws s3 rb s3://my-bucket
aws s3 rb s3://my-bucket --force  # Remove with contents
```

### Object Operations

```bash
# Upload file
aws s3 cp file.txt s3://my-bucket/
aws s3 cp file.txt s3://my-bucket/path/file.txt

# Upload directory
aws s3 cp ./mydir s3://my-bucket/mydir --recursive

# Download file
aws s3 cp s3://my-bucket/file.txt ./
aws s3 cp s3://my-bucket/file.txt ./downloaded.txt

# Download directory
aws s3 cp s3://my-bucket/mydir ./ --recursive

# Sync directory
aws s3 sync ./local-dir s3://my-bucket/remote-dir
aws s3 sync s3://my-bucket/remote-dir ./local-dir

# Delete file
aws s3 rm s3://my-bucket/file.txt

# Delete directory
aws s3 rm s3://my-bucket/path/ --recursive

# Move file
aws s3 mv s3://my-bucket/old.txt s3://my-bucket/new.txt

# List object versions
aws s3api list-object-versions --bucket my-bucket
```

### Bucket Configuration

```bash
# Enable versioning
aws s3api put-bucket-versioning --bucket my-bucket --versioning-configuration Status=Enabled

# Get bucket versioning
aws s3api get-bucket-versioning --bucket my-bucket

# Set bucket policy
aws s3api put-bucket-policy --bucket my-bucket --policy file://policy.json

# Get bucket policy
aws s3api get-bucket-policy --bucket my-bucket

# Enable static website hosting
aws s3 website s3://my-bucket --index-document index.html --error-document error.html

# Enable server-side encryption
aws s3api put-bucket-encryption --bucket my-bucket --server-side-encryption-configuration '{
  "Rules": [
    {
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }
  ]
}'

# Generate presigned URL (expires in 3600 seconds)
aws s3 presign s3://my-bucket/file.txt --expires-in 3600
```

## VPC (Virtual Private Cloud)

### VPC Management

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# List VPCs
aws ec2 describe-vpcs

# Delete VPC
aws ec2 delete-vpc --vpc-id vpc-0123456789abcdef0

# Create subnet
aws ec2 create-subnet --vpc-id vpc-0123456789abcdef0 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a

# List subnets
aws ec2 describe-subnets

# Create internet gateway
aws ec2 create-internet-gateway

# Attach internet gateway to VPC
aws ec2 attach-internet-gateway --vpc-id vpc-0123456789abcdef0 --internet-gateway-id igw-0123456789abcdef0

# Create route table
aws ec2 create-route-table --vpc-id vpc-0123456789abcdef0

# Create route
aws ec2 create-route --route-table-id rtb-0123456789abcdef0 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0123456789abcdef0

# Associate route table with subnet
aws ec2 associate-route-table --subnet-id subnet-0123456789abcdef0 --route-table-id rtb-0123456789abcdef0
```

## RDS (Relational Database Service)

### DB Instance Management

```bash
# Create DB instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password mypassword \
  --allocated-storage 20

# List DB instances
aws rds describe-db-instances

# Describe specific DB instance
aws rds describe-db-instances --db-instance-identifier mydb

# Modify DB instance
aws rds modify-db-instance --db-instance-identifier mydb --allocated-storage 30 --apply-immediately

# Start DB instance
aws rds start-db-instance --db-instance-identifier mydb

# Stop DB instance
aws rds stop-db-instance --db-instance-identifier mydb

# Reboot DB instance
aws rds reboot-db-instance --db-instance-identifier mydb

# Delete DB instance
aws rds delete-db-instance --db-instance-identifier mydb --skip-final-snapshot

# Create DB snapshot
aws rds create-db-snapshot --db-instance-identifier mydb --db-snapshot-identifier mydb-snapshot-1

# List DB snapshots
aws rds describe-db-snapshots

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot --db-instance-identifier mydb-restored --db-snapshot-identifier mydb-snapshot-1
```

## Lambda

### Function Management

```bash
# Create function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler index.handler \
  --zip-file fileb://function.zip

# List functions
aws lambda list-functions

# Get function
aws lambda get-function --function-name my-function

# Invoke function
aws lambda invoke \
  --function-name my-function \
  --payload '{"key":"value"}' \
  response.json

# Update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# Update function configuration
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 30 \
  --memory-size 256

# Delete function
aws lambda delete-function --function-name my-function

# Add environment variables
aws lambda update-function-configuration \
  --function-name my-function \
  --environment Variables={KEY1=value1,KEY2=value2}
```

### Example Lambda Function (Python)

```python
import json

def handler(event, context):
    print(f"Received event: {json.dumps(event)}")

    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Hello from Lambda!',
            'input': event
        })
    }
```

## DynamoDB

### Table Operations

```bash
# Create table
aws dynamodb create-table \
  --table-name Users \
  --attribute-definitions \
    AttributeName=UserId,AttributeType=S \
  --key-schema \
    AttributeName=UserId,KeyType=HASH \
  --provisioned-throughput \
    ReadCapacityUnits=5,WriteCapacityUnits=5

# List tables
aws dynamodb list-tables

# Describe table
aws dynamodb describe-table --table-name Users

# Delete table
aws dynamodb delete-table --table-name Users
```

### Item Operations

```bash
# Put item
aws dynamodb put-item \
  --table-name Users \
  --item '{
    "UserId": {"S": "user123"},
    "Name": {"S": "John Doe"},
    "Email": {"S": "john@example.com"}
  }'

# Get item
aws dynamodb get-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}'

# Query items
aws dynamodb query \
  --table-name Users \
  --key-condition-expression "UserId = :userId" \
  --expression-attribute-values '{":userId": {"S": "user123"}}'

# Scan table
aws dynamodb scan --table-name Users

# Update item
aws dynamodb update-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}' \
  --update-expression "SET Email = :email" \
  --expression-attribute-values '{":email": {"S": "newemail@example.com"}}'

# Delete item
aws dynamodb delete-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}'

# Batch write items
aws dynamodb batch-write-item --request-items file://items.json
```

## CloudFormation

### Stack Management

```bash
# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=my-key

# List stacks
aws cloudformation list-stacks

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Validate template
aws cloudformation validate-template --template-body file://template.yaml

# Get stack events
aws cloudformation describe-stack-events --stack-name my-stack

# Get stack resources
aws cloudformation describe-stack-resources --stack-name my-stack
```

### Example CloudFormation Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple EC2 instance

Parameters:
  KeyName:
    Description: EC2 Key Pair
    Type: AWS::EC2::KeyPair::KeyName

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      KeyName: !Ref KeyName
      Tags:
        - Key: Name
          Value: MyInstance

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref MyEC2Instance
  PublicIP:
    Description: Public IP
    Value: !GetAtt MyEC2Instance.PublicIp
```

## ECS & EKS

### ECS (Elastic Container Service)

```bash
# Create cluster
aws ecs create-cluster --cluster-name my-cluster

# List clusters
aws ecs list-clusters

# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# List task definitions
aws ecs list-task-definitions

# Run task
aws ecs run-task --cluster my-cluster --task-definition my-task

# Create service
aws ecs create-service \
  --cluster my-cluster \
  --service-name my-service \
  --task-definition my-task \
  --desired-count 2

# List services
aws ecs list-services --cluster my-cluster

# Update service
aws ecs update-service \
  --cluster my-cluster \
  --service my-service \
  --desired-count 3

# Delete service
aws ecs delete-service --cluster my-cluster --service my-service
```

### EKS (Elastic Kubernetes Service)

```bash
# Create cluster (use eksctl for easier setup)
eksctl create cluster --name my-cluster --region us-east-1

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-east-1

# List clusters
aws eks list-clusters

# Describe cluster
aws eks describe-cluster --name my-cluster

# Delete cluster
eksctl delete cluster --name my-cluster
```

## Common Patterns

### Multi-Region Deployment

```yaml
Architecture:
  Region 1 (Primary):
    - Application Load Balancer
    - Auto Scaling Group
    - RDS Primary
    - S3 bucket (versioned)

  Region 2 (Secondary):
    - Application Load Balancer
    - Auto Scaling Group
    - RDS Read Replica
    - S3 bucket (cross-region replication)

  Global:
    - Route 53 (Geo-routing/Failover)
    - CloudFront CDN
```

### Serverless Architecture

```yaml
Architecture:
  Client:
    ↓
  CloudFront + S3 (Static content)
    ↓
  API Gateway
    ↓
  Lambda Functions
    ↓
  DynamoDB / RDS Aurora Serverless
    ↓
  S3 (Object storage)

Benefits:
  - No server management
  - Auto-scaling
  - Pay-per-use
  - High availability
```

### Cost Optimization

```yaml
Strategies:
  1. Right-sizing:
     - Use appropriate instance types
     - Monitor CloudWatch metrics
     - Downsize underutilized resources

  2. Reserved Instances:
     - 1-year or 3-year commitment
     - Up to 75% savings
     - For predictable workloads

  3. Spot Instances:
     - Up to 90% savings
     - For fault-tolerant workloads
     - Stateless applications

  4. Auto Scaling:
     - Scale down during low traffic
     - Schedule scaling
     - Target tracking

  5. S3 Storage Classes:
     - Standard (frequent access)
     - Infrequent Access (IA)
     - Glacier (archival)
     - Intelligent-Tiering

  6. Delete Unused Resources:
     - Unattached EBS volumes
     - Old snapshots
     - Unused Elastic IPs
```

## Additional Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS CLI Command Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Free Tier](https://aws.amazon.com/free/)

---

*Last updated: 2025-11-16*
