
```
Why AWS?
  ✅ 33% of cloud market (largest by far)
  ✅ Used by: Netflix, Airbnb, NASA, Slack, Reddit
  ✅ Most backend job listings require AWS knowledge
  ✅ Free tier = learn for free
  ✅ Every cloud concept you learn here applies to GCP/Azure

What we'll cover:
┌─────────────────────────────────────────────────────────────┐
│  IAM       → Who can do what (security foundation)         │
│  EC2       → Virtual machines (servers in the cloud)       │
│  RDS       → Managed PostgreSQL                            │
│  S3        → File/object storage                           │
│  ElastiCache→ Managed Redis                                │
│  SQS       → Message queues (Celery backend)               │
│  ECR       → Docker image registry                         │
│  ECS       → Run Docker containers                         │
│  Lambda    → Serverless functions                          │
│  CloudWatch→ Monitoring and logs                           │
│  Route 53  → DNS management                               │
│  ALB       → Load balancer                                │
└─────────────────────────────────────────────────────────────┘

By end of this phase:
  Your FastAPI app runs on AWS
  With managed PostgreSQL + Redis
  Behind a load balancer
  With auto-scaling
  Logs in CloudWatch
  Files in S3
```

---

## Setup

Bash

```
# Install AWS CLI
# Mac:
brew install awscli

# Linux:
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
    -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Windows: download from aws.amazon.com/cli

# Verify
aws --version
# aws-cli/2.x.x Python/3.x.x

# Install boto3 (Python SDK)
pip install boto3

# Create AWS account:
# 1. Go to aws.amazon.com
# 2. Create account (credit card needed, but free tier available)
# 3. Do NOT use root account for anything — create IAM user
```

---

## 1. 🔐 IAM — Identity and Access Management

### The Foundation of AWS Security

text

```
IAM answers: "WHO can do WHAT to WHICH resources"

WHO  = Users, Groups, Roles, Services
WHAT = Actions (CreateBucket, DescribeInstances, etc.)
WHICH= Resources (specific S3 bucket, specific EC2, or *)

Principal of Least Privilege:
  Grant ONLY the permissions needed.
  Nothing more.

Examples:
  ✅ EC2 instance can read from specific S3 bucket
  ✅ Lambda can write to specific DynamoDB table
  ✅ Developer can deploy to ECS but not delete databases
  ❌ EC2 instance with admin access (bad practice!)
```

### Setting Up IAM Properly

Bash

```
# ─────────────────────────────────────────
# NEVER USE ROOT ACCOUNT
# ─────────────────────────────────────────
# Root account has unlimited power.
# Use it ONLY to create your first admin IAM user.
# Then LOCK THE ROOT CREDENTIALS and never use again.

# Step 1: Log into AWS console as root
# Step 2: Go to IAM → Users → Create User

# Step 3: Configure AWS CLI with IAM user credentials
aws configure
# AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name [None]: us-east-1
# Default output format [None]: json

# Verify
aws sts get-caller-identity
# {
#     "UserId": "AIDIOSFODNN7EXAMPLE",
#     "Account": "123456789012",
#     "Arn": "arn:aws:iam::123456789012:user/myuser"
# }

# Multiple profiles
aws configure --profile staging
aws configure --profile production
aws --profile staging s3 ls        # use specific profile
export AWS_PROFILE=staging          # set default profile
```

### IAM Policies — The Permission System

JSON

```
// IAM Policy example — allows specific S3 operations on specific bucket
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowS3ReadWrite",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::my-app-bucket",
                "arn:aws:s3:::my-app-bucket/*"
            ]
        },
        {
            "Sid": "DenyDelete",
            "Effect": "Deny",
            "Action": "s3:DeleteObject",
            "Resource": "arn:aws:s3:::my-app-bucket/important/*"
        }
    ]
}
```

Python

```
# boto3_demo.py — Python AWS SDK examples
import boto3
from botocore.exceptions import ClientError
import json


# ─────────────────────────────────────────
# CREATE IAM ROLES for your services
# ─────────────────────────────────────────
iam = boto3.client("iam")


def create_ec2_role_for_app():
    """
    Create an IAM role that EC2 instances can assume.
    Gives your app permissions without hardcoding credentials.
    """

    # Trust policy: who can assume this role?
    trust_policy = {
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "ec2.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }

    # Create the role
    role = iam.create_role(
        RoleName="MyAppEC2Role",
        AssumeRolePolicyDocument=json.dumps(trust_policy),
        Description="Role for MyApp EC2 instances"
    )

    # Attach permission policies to the role
    policies_to_attach = [
        # Allow reading from S3
        {
            "PolicyName": "S3ReadWrite",
            "PolicyDocument": json.dumps({
                "Version": "2012-10-17",
                "Statement": [{
                    "Effect": "Allow",
                    "Action": [
                        "s3:GetObject", "s3:PutObject",
                        "s3:DeleteObject", "s3:ListBucket"
                    ],
                    "Resource": [
                        "arn:aws:s3:::my-app-bucket",
                        "arn:aws:s3:::my-app-bucket/*"
                    ]
                }]
            })
        },
        # Allow CloudWatch logging
        {
            "PolicyName": "CloudWatchLogs",
            "PolicyDocument": json.dumps({
                "Version": "2012-10-17",
                "Statement": [{
                    "Effect": "Allow",
                    "Action": [
                        "logs:CreateLogGroup",
                        "logs:CreateLogStream",
                        "logs:PutLogEvents",
                        "logs:DescribeLogStreams"
                    ],
                    "Resource": "arn:aws:logs:*:*:*"
                }]
            })
        },
        # Allow reading secrets
        {
            "PolicyName": "SecretsManagerRead",
            "PolicyDocument": json.dumps({
                "Version": "2012-10-17",
                "Statement": [{
                    "Effect": "Allow",
                    "Action": [
                        "secretsmanager:GetSecretValue",
                        "secretsmanager:DescribeSecret"
                    ],
                    "Resource": (
                        "arn:aws:secretsmanager:us-east-1:123456789012:secret:"
                        "myapp/*"
                    )
                }]
            })
        }
    ]

    for policy in policies_to_attach:
        iam.put_role_policy(
            RoleName="MyAppEC2Role",
            PolicyName=policy["PolicyName"],
            PolicyDocument=policy["PolicyDocument"]
        )

    # Create instance profile (wrapper that EC2 can use)
    iam.create_instance_profile(InstanceProfileName="MyAppEC2Role")
    iam.add_role_to_instance_profile(
        InstanceProfileName="MyAppEC2Role",
        RoleName="MyAppEC2Role"
    )

    print(f"Created role: {role['Role']['Arn']}")
    return role


# create_ec2_role_for_app()
```

---

## 2. 🖥️ EC2 — Elastic Compute Cloud

### What is EC2?

text

```
EC2 = Virtual machines in the cloud.
      You choose: CPU, RAM, storage, OS.
      Billed by hour or second.

Instance types:
  t3.micro   = 2 vCPU, 1GB RAM  → free tier, dev/test
  t3.small   = 2 vCPU, 2GB RAM  → small apps
  t3.medium  = 2 vCPU, 4GB RAM  → medium apps
  t3.large   = 2 vCPU, 8GB RAM  → larger apps
  c5.large   = 2 vCPU, 4GB RAM  → compute optimized
  m5.large   = 2 vCPU, 8GB RAM  → general purpose
  r5.large   = 2 vCPU, 16GB RAM → memory optimized

Key concepts:
  AMI:          Amazon Machine Image — OS + software snapshot
  Security Group: Firewall for your instance
  Key pair:     SSH access
  Elastic IP:   Static IP address that doesn't change on restart
  EBS volume:   Persistent disk storage (survives instance stop)
  User data:    Script that runs when instance first starts
```

### Launching an EC2 Instance

Python

```
# ec2_demo.py

import boto3
import time

ec2 = boto3.client("ec2", region_name="us-east-1")


def create_security_group(vpc_id: str, name: str) -> str:
    """Create a security group for our app server."""

    response = ec2.create_security_group(
        GroupName=name,
        Description=f"Security group for {name}",
        VpcId=vpc_id,
        TagSpecifications=[{
            "ResourceType": "security-group",
            "Tags": [
                {"Key": "Name", "Value": name},
                {"Key": "Environment", "Value": "production"},
            ]
        }]
    )

    sg_id = response["GroupId"]
    print(f"Created security group: {sg_id}")

    # Add inbound rules
    ec2.authorize_security_group_ingress(
        GroupId=sg_id,
        IpPermissions=[
            # SSH from specific IP only (your office/home)
            {
                "IpProtocol": "tcp",
                "FromPort": 22,
                "ToPort": 22,
                "IpRanges": [{"CidrIp": "YOUR_IP/32", "Description": "My IP"}]
            },
            # HTTP from anywhere (Nginx handles HTTPS redirect)
            {
                "IpProtocol": "tcp",
                "FromPort": 80,
                "ToPort": 80,
                "IpRanges": [{"CidrIp": "0.0.0.0/0"}]
            },
            # HTTPS from anywhere
            {
                "IpProtocol": "tcp",
                "FromPort": 443,
                "ToPort": 443,
                "IpRanges": [{"CidrIp": "0.0.0.0/0"}]
            },
            # App port - only from load balancer security group
            {
                "IpProtocol": "tcp",
                "FromPort": 8000,
                "ToPort": 8000,
                "UserIdGroupPairs": [{"GroupId": "sg-loadbalancer-id"}]
            }
        ]
    )

    return sg_id


def create_user_data_script() -> str:
    """
    Script that runs when EC2 instance first starts.
    Use this to install Docker, set up your app, etc.
    """
    return """#!/bin/bash
set -e

# Update system
apt-get update -y
apt-get upgrade -y

# Install Docker
apt-get install -y ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) \
    signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update -y
apt-get install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

# Add ubuntu user to docker group
usermod -aG docker ubuntu

# Install AWS CLI (for ECR login)
apt-get install -y awscli

# Clone app and start
mkdir -p /opt/myapp
cd /opt/myapp

# Login to ECR
aws ecr get-login-password --region us-east-1 | \
    docker login --username AWS \
    --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Pull and start app
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker compose up -d

echo "Setup complete!" >> /var/log/user-data.log
"""


def launch_instance(
    security_group_id: str,
    subnet_id: str,
    instance_type: str = "t3.micro"
) -> dict:
    """Launch an EC2 instance."""

    import base64
    user_data = base64.b64encode(
        create_user_data_script().encode()
    ).decode()

    response = ec2.run_instances(
        ImageId="ami-0c7217cdde317cfec",  # Ubuntu 22.04 us-east-1
        InstanceType=instance_type,
        MinCount=1,
        MaxCount=1,
        KeyName="my-keypair",             # your SSH key
        SecurityGroupIds=[security_group_id],
        SubnetId=subnet_id,
        UserData=user_data,

        # IAM role for permissions (no hardcoded credentials!)
        IamInstanceProfile={"Name": "MyAppEC2Role"},

        # Root volume
        BlockDeviceMappings=[{
            "DeviceName": "/dev/sda1",
            "Ebs": {
                "VolumeSize": 30,          # 30GB
                "VolumeType": "gp3",       # general purpose SSD
                "DeleteOnTermination": True,
                "Encrypted": True,         # always encrypt!
            }
        }],

        # Enable detailed monitoring
        Monitoring={"Enabled": True},

        TagSpecifications=[{
            "ResourceType": "instance",
            "Tags": [
                {"Key": "Name", "Value": "myapp-production"},
                {"Key": "Environment", "Value": "production"},
                {"Key": "Project", "Value": "myapp"},
            ]
        }]
    )

    instance = response["Instances"][0]
    instance_id = instance["InstanceId"]

    print(f"Launched instance: {instance_id}")
    print("Waiting for instance to be running...")

    # Wait for instance to be running
    waiter = ec2.get_waiter("instance_running")
    waiter.wait(InstanceIds=[instance_id])

    # Get public IP
    describe = ec2.describe_instances(InstanceIds=[instance_id])
    public_ip = describe["Reservations"][0]["Instances"][0].get(
        "PublicIpAddress", "No public IP"
    )

    print(f"Instance running! Public IP: {public_ip}")
    return instance
```

### EC2 Operations

Bash

```
# ─────────────────────────────────────────
# AWS CLI EC2 COMMANDS
# ─────────────────────────────────────────

# List instances
aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].{ID:InstanceId,IP:PublicIpAddress,Type:InstanceType,Name:Tags[?Key=='Name']|[0].Value}" \
    --output table

# Start/stop instances
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Get instance public IP
aws ec2 describe-instances \
    --instance-ids i-1234567890abcdef0 \
    --query "Reservations[0].Instances[0].PublicIpAddress" \
    --output text

# Create Elastic IP (static IP)
aws ec2 allocate-address --domain vpc
# Returns: AllocationId, PublicIp

# Associate with instance
aws ec2 associate-address \
    --instance-id i-1234567890abcdef0 \
    --allocation-id eipalloc-abcdef1234567890a

# SSH to instance
ssh -i ~/.ssh/my-keypair.pem ubuntu@YOUR_ELASTIC_IP
```

---

## 3. 🗄️ RDS — Managed PostgreSQL

### Why RDS Instead of Self-Managed PostgreSQL?

text

```
Self-managed (on EC2):
  ❌ You manage: backups, patches, replication, failover
  ❌ Disk fills up → you fix it
  ❌ PostgreSQL crashes → you debug
  ❌ No automatic failover
  ❌ You wake up at 3am for database issues

RDS (managed):
  ✅ Automated backups (point-in-time recovery)
  ✅ Automated patches (configurable window)
  ✅ Multi-AZ: automatic failover in 60 seconds
  ✅ Read replicas: scale read traffic
  ✅ Storage auto-scaling
  ✅ CloudWatch metrics built-in
  ✅ You focus on your app, not database ops
  
Cost: ~$15-50/month for production-ready setup
Worth it for: anyone who values sleep.
```

Python

```
# rds_demo.py

import boto3

rds = boto3.client("rds", region_name="us-east-1")


def create_db_subnet_group(subnet_ids: list, name: str) -> str:
    """DB must be in private subnets (not accessible from internet)."""
    response = rds.create_db_subnet_group(
        DBSubnetGroupName=name,
        DBSubnetGroupDescription=f"Subnet group for {name}",
        SubnetIds=subnet_ids,
        Tags=[{"Key": "Project", "Value": "myapp"}]
    )
    return response["DBSubnetGroup"]["DBSubnetGroupName"]


def create_rds_instance(
    db_name: str = "myapp_db",
    username: str = "myapp_admin",
    password: str = "use-secrets-manager-in-production!",
) -> dict:
    """Create a production-ready RDS PostgreSQL instance."""

    response = rds.create_db_instance(
        DBInstanceIdentifier="myapp-postgres",
        DBInstanceClass="db.t3.micro",       # free tier eligible
        Engine="postgres",
        EngineVersion="16.1",
        DBName=db_name,
        MasterUsername=username,
        MasterUserPassword=password,
        AllocatedStorage=20,                  # 20 GB
        MaxAllocatedStorage=100,              # auto-scale up to 100GB
        StorageType="gp3",                    # SSD
        StorageEncrypted=True,                # ALWAYS encrypt!

        # High Availability
        MultiAZ=True,                         # standby in another AZ
        # AutoMinorVersionUpgrade=True,       # auto patch minor versions

        # Networking
        DBSubnetGroupName="myapp-db-subnet-group",
        VpcSecurityGroupIds=["sg-your-db-sg"],
        PubliclyAccessible=False,             # NEVER public!

        # Backup
        BackupRetentionPeriod=7,              # keep 7 days of backups
        PreferredBackupWindow="02:00-03:00",  # 2-3am UTC
        PreferredMaintenanceWindow="sun:03:00-sun:04:00",

        # Parameters
        DBParameterGroupName="default.postgres16",

        # Deletion protection (important!)
        DeletionProtection=True,

        Tags=[
            {"Key": "Name", "Value": "myapp-postgres"},
            {"Key": "Environment", "Value": "production"},
        ]
    )

    db = response["DBInstance"]
    print(f"Creating RDS instance: {db['DBInstanceIdentifier']}")
    print(f"Status: {db['DBInstanceStatus']}")

    # Wait for it to be available (takes 5-15 minutes)
    print("Waiting for RDS to be available (this takes a while)...")
    waiter = rds.get_waiter("db_instance_available")
    waiter.wait(DBInstanceIdentifier="myapp-postgres")

    # Get the endpoint
    response = rds.describe_db_instances(
        DBInstanceIdentifier="myapp-postgres"
    )
    endpoint = response["DBInstances"][0]["Endpoint"]
    print(f"RDS Available!")
    print(f"Endpoint: {endpoint['Address']}:{endpoint['Port']}")

    return response["DBInstances"][0]


def create_db_snapshot(instance_id: str, snapshot_id: str):
    """Create a manual snapshot before major changes."""
    rds.create_db_snapshot(
        DBInstanceIdentifier=instance_id,
        DBSnapshotIdentifier=snapshot_id,
        Tags=[{"Key": "Type", "Value": "pre-deployment"}]
    )
    print(f"Creating snapshot: {snapshot_id}")


def restore_from_snapshot(snapshot_id: str, new_instance_id: str):
    """Restore database from snapshot (disaster recovery)."""
    rds.restore_db_instance_from_db_snapshot(
        DBInstanceIdentifier=new_instance_id,
        DBSnapshotIdentifier=snapshot_id,
        DBInstanceClass="db.t3.micro",
        MultiAZ=False,           # single AZ for restored instance
        PubliclyAccessible=False,
    )
```

Bash

```
# ─────────────────────────────────────────
# RDS CLI COMMANDS
# ─────────────────────────────────────────

# List databases
aws rds describe-db-instances \
    --query "DBInstances[*].{ID:DBInstanceIdentifier,Status:DBInstanceStatus,Endpoint:Endpoint.Address}" \
    --output table

# Create snapshot
aws rds create-db-snapshot \
    --db-instance-identifier myapp-postgres \
    --db-snapshot-identifier myapp-backup-20240115

# Get connection info
aws rds describe-db-instances \
    --db-instance-identifier myapp-postgres \
    --query "DBInstances[0].Endpoint" \
    --output json

# Connect from EC2 (using psql)
# Database is in private subnet — connect via EC2 (bastion)
# On your EC2 instance:
psql -h myapp-postgres.abc123.us-east-1.rds.amazonaws.com \
     -U myapp_admin \
     -d myapp_db

# Or SSH tunnel from your laptop:
ssh -L 5433:myapp-postgres.abc123.us-east-1.rds.amazonaws.com:5432 \
    ubuntu@YOUR_EC2_IP

# Then from laptop:
psql -h localhost -p 5433 -U myapp_admin -d myapp_db
```

---

## 4. 🪣 S3 — Object Storage

### What is S3?

text

```
S3 = Simple Storage Service
     Store any file: images, videos, PDFs, backups, logs
     Virtually unlimited storage
     99.999999999% durability (11 nines!)
     Cheap: $0.023/GB/month
     
Use cases for backends:
  ✅ User uploads (profile pictures, documents)
  ✅ Static assets (CSS, JS, images)
  ✅ Database backups
  ✅ Log archiving
  ✅ Large data exports (CSV, PDF reports)
  ✅ Application artifacts (Docker images, build files)
  
Key concepts:
  Bucket:   Container for objects (like a folder)
  Object:   A file + metadata (key + value + metadata)
  Key:      File path within bucket (e.g., "users/42/avatar.jpg")
  ACL:      Who can access the object
  Presigned URL: Temporary URL for private files
```

Python

```
# s3_demo.py
import boto3
from botocore.exceptions import ClientError
from botocore.config import Config
import os
from pathlib import Path
from typing import Optional
import mimetypes


s3 = boto3.client(
    "s3",
    region_name="us-east-1",
    config=Config(
        retries={"max_attempts": 3, "mode": "adaptive"},
        multipart_threshold=1024 * 25,      # 25MB threshold
        multipart_chunksize=1024 * 25,      # 25MB chunks
    )
)


# ─────────────────────────────────────────
# BUCKET SETUP
# ─────────────────────────────────────────
def create_bucket(bucket_name: str, region: str = "us-east-1") -> None:
    """Create an S3 bucket with proper configuration."""

    # Create bucket
    if region == "us-east-1":
        s3.create_bucket(Bucket=bucket_name)
    else:
        s3.create_bucket(
            Bucket=bucket_name,
            CreateBucketConfiguration={"LocationConstraint": region}
        )

    # Block ALL public access (then use presigned URLs for private files)
    s3.put_public_access_block(
        Bucket=bucket_name,
        PublicAccessBlockConfiguration={
            "BlockPublicAcls": True,
            "IgnorePublicAcls": True,
            "BlockPublicPolicy": True,
            "RestrictPublicBuckets": True,
        }
    )

    # Enable versioning (recover from accidental deletes)
    s3.put_bucket_versioning(
        Bucket=bucket_name,
        VersioningConfiguration={"Status": "Enabled"}
    )

    # Enable server-side encryption
    s3.put_bucket_encryption(
        Bucket=bucket_name,
        ServerSideEncryptionConfiguration={
            "Rules": [{
                "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "AES256"
                },
                "BucketKeyEnabled": True
            }]
        }
    )

    # Lifecycle policy: delete old versions after 90 days
    s3.put_bucket_lifecycle_configuration(
        Bucket=bucket_name,
        LifecycleConfiguration={
            "Rules": [{
                "ID": "DeleteOldVersions",
                "Status": "Enabled",
                "NoncurrentVersionExpiration": {
                    "NoncurrentDays": 90
                }
            }]
        }
    )

    print(f"Bucket created: {bucket_name}")


# ─────────────────────────────────────────
# FILE OPERATIONS
# ─────────────────────────────────────────
class S3Storage:
    """Production S3 storage service."""

    def __init__(self, bucket_name: str, region: str = "us-east-1"):
        self.bucket = bucket_name
        self.region = region
        self.s3 = boto3.client("s3", region_name=region)

    def upload_file(
        self,
        local_path: str,
        s3_key: str,
        content_type: Optional[str] = None,
        metadata: Optional[dict] = None,
        make_public: bool = False,
    ) -> str:
        """Upload a file to S3."""
        # Detect content type
        if not content_type:
            content_type, _ = mimetypes.guess_type(local_path)
            content_type = content_type or "application/octet-stream"

        extra_args = {
            "ContentType": content_type,
            "ServerSideEncryption": "AES256",  # encrypt in transit
        }

        if metadata:
            extra_args["Metadata"] = {
                k: str(v) for k, v in metadata.items()
            }

        if make_public:
            extra_args["ACL"] = "public-read"

        # Cache control for static assets
        if content_type.startswith("image/"):
            extra_args["CacheControl"] = "max-age=31536000"  # 1 year

        # Upload (automatically handles multipart for large files)
        self.s3.upload_file(
            local_path,
            self.bucket,
            s3_key,
            ExtraArgs=extra_args
        )

        url = (f"https://{self.bucket}.s3.{self.region}"
               f".amazonaws.com/{s3_key}")
        print(f"Uploaded: {s3_key}")
        return url

    def upload_bytes(
        self,
        data: bytes,
        s3_key: str,
        content_type: str = "application/octet-stream",
    ) -> str:
        """Upload bytes directly to S3."""
        self.s3.put_object(
            Bucket=self.bucket,
            Key=s3_key,
            Body=data,
            ContentType=content_type,
            ServerSideEncryption="AES256",
        )
        return f"s3://{self.bucket}/{s3_key}"

    def download_file(self, s3_key: str, local_path: str) -> None:
        """Download file from S3."""
        os.makedirs(os.path.dirname(local_path), exist_ok=True)
        self.s3.download_file(self.bucket, s3_key, local_path)

    def get_bytes(self, s3_key: str) -> bytes:
        """Get file contents as bytes."""
        response = self.s3.get_object(Bucket=self.bucket, Key=s3_key)
        return response["Body"].read()

    def delete(self, s3_key: str) -> None:
        """Delete a file from S3."""
        self.s3.delete_object(Bucket=self.bucket, Key=s3_key)

    def exists(self, s3_key: str) -> bool:
        """Check if a file exists in S3."""
        try:
            self.s3.head_object(Bucket=self.bucket, Key=s3_key)
            return True
        except ClientError as e:
            if e.response["Error"]["Code"] == "404":
                return False
            raise

    def list_files(self, prefix: str = "", max_keys: int = 1000) -> list:
        """List files in S3 bucket."""
        response = self.s3.list_objects_v2(
            Bucket=self.bucket,
            Prefix=prefix,
            MaxKeys=max_keys
        )
        return [obj["Key"] for obj in response.get("Contents", [])]

    # ─────────────────────────────────────────
    # PRESIGNED URLS — key pattern for private files
    # ─────────────────────────────────────────
    def generate_presigned_url(
        self,
        s3_key: str,
        expiration: int = 3600,   # seconds
        operation: str = "get_object"
    ) -> str:
        """
        Generate a temporary URL for accessing a private file.

        Use cases:
          - User wants to download their own invoice PDF
          - Share a private image temporarily
          - Let client upload directly to S3 (bypass your server!)

        The URL expires after 'expiration' seconds.
        No AWS credentials needed by the user.
        """
        url = self.s3.generate_presigned_url(
            ClientMethod=operation,
            Params={
                "Bucket": self.bucket,
                "Key": s3_key,
            },
            ExpiresIn=expiration
        )
        return url

    def generate_presigned_upload_url(
        self,
        s3_key: str,
        content_type: str,
        max_size_bytes: int = 10 * 1024 * 1024,  # 10MB
        expiration: int = 300,  # 5 minutes
    ) -> dict:
        """
        Generate a presigned POST URL for DIRECT upload to S3.

        Client uploads directly to S3 — your server never touches the file!
        Benefits:
          ✅ No bandwidth cost on your server
          ✅ Faster (direct upload)
          ✅ You control size/type constraints

        Returns URL and fields needed for the upload.
        """
        response = self.s3.generate_presigned_post(
            Bucket=self.bucket,
            Key=s3_key,
            Fields={
                "Content-Type": content_type,
            },
            Conditions=[
                {"Content-Type": content_type},
                ["content-length-range", 1, max_size_bytes],
            ],
            ExpiresIn=expiration
        )
        return response


# ─────────────────────────────────────────
# FASTAPI INTEGRATION
# ─────────────────────────────────────────
# In your FastAPI app:
from fastapi import UploadFile, File, HTTPException
from fastapi import APIRouter
import uuid

storage = S3Storage("my-app-bucket")
router = APIRouter()


@router.post("/upload/avatar")
async def upload_avatar(
    file: UploadFile = File(...),
    current_user = None,  # from auth dependency
):
    """Upload user avatar to S3."""

    # Validate file type
    allowed_types = {"image/jpeg", "image/png", "image/webp"}
    if file.content_type not in allowed_types:
        raise HTTPException(400, f"File type not allowed: {file.content_type}")

    # Validate file size (read into memory)
    content = await file.read()
    if len(content) > 5 * 1024 * 1024:  # 5MB limit
        raise HTTPException(413, "File too large. Max 5MB")

    # Generate unique key
    ext = file.filename.rsplit(".", 1)[-1].lower()
    key = f"avatars/{current_user.id}/{uuid.uuid4()}.{ext}"

    # Upload to S3
    storage.upload_bytes(content, key, file.content_type)

    # Return presigned URL (valid for 1 hour)
    url = storage.generate_presigned_url(key, expiration=3600)

    return {"avatar_url": url, "key": key}


@router.get("/files/{file_key}/download")
async def get_download_url(
    file_key: str,
    current_user = None,
):
    """Get temporary download URL for a private file."""
    # Verify user owns this file
    if not file_key.startswith(f"uploads/{current_user.id}/"):
        raise HTTPException(403, "Access denied")

    url = storage.generate_presigned_url(
        file_key,
        expiration=300  # 5 minutes
    )
    return {"download_url": url, "expires_in": 300}


@router.post("/upload/direct-url")
async def get_upload_url(
    filename: str,
    content_type: str,
    current_user = None,
):
    """
    Get a presigned URL for direct browser upload to S3.
    Client uploads directly — saves your server bandwidth.
    """
    # Validate content type
    if content_type not in {"image/jpeg", "image/png", "application/pdf"}:
        raise HTTPException(400, "File type not supported")

    ext = filename.rsplit(".", 1)[-1]
    key = f"uploads/{current_user.id}/{uuid.uuid4()}.{ext}"

    presigned = storage.generate_presigned_upload_url(
        key, content_type, max_size_bytes=20 * 1024 * 1024
    )

    return {
        "upload_url": presigned["url"],
        "fields": presigned["fields"],
        "key": key,
        "expires_in": 300
    }
```

---

## 5. ⚡ ElastiCache — Managed Redis

Python

```
# elasticache_demo.py
import boto3

elasticache = boto3.client("elasticache", region_name="us-east-1")


def create_redis_cluster(
    cluster_name: str = "myapp-redis",
    node_type: str = "cache.t3.micro",  # free tier
) -> dict:
    """Create a managed Redis cluster."""

    response = elasticache.create_cache_cluster(
        CacheClusterId=cluster_name,
        Engine="redis",
        EngineVersion="7.0",
        CacheNodeType=node_type,
        NumCacheNodes=1,             # single node (use replication group for HA)

        # Networking
        CacheSubnetGroupName="myapp-cache-subnet-group",
        SecurityGroupIds=["sg-your-cache-sg"],

        # No public access! Only internal network.
        # VPC subnet group puts it in private subnet.

        # Backup
        SnapshotRetentionLimit=1,    # 1 day of backups

        # Encryption
        AtRestEncryptionEnabled=True,
        TransitEncryptionEnabled=True,
        AuthToken="your-strong-redis-password",  # use Secrets Manager!

        Tags=[
            {"Key": "Name", "Value": cluster_name},
            {"Key": "Environment", "Value": "production"},
        ]
    )

    cluster = response["CacheCluster"]
    print(f"Creating Redis cluster: {cluster['CacheClusterId']}")

    # Wait for it to be available
    waiter = elasticache.get_waiter("cache_cluster_available")
    waiter.wait(CacheClusterId=cluster_name)

    # Get endpoint
    response = elasticache.describe_cache_clusters(
        CacheClusterId=cluster_name,
        ShowCacheNodeInfo=True
    )
    node = response["CacheClusters"][0]["CacheNodes"][0]
    endpoint = node["Endpoint"]
    print(f"Redis available: {endpoint['Address']}:{endpoint['Port']}")

    return response["CacheClusters"][0]


# Redis connection string for your app:
# redis://:password@your-cluster.abc123.0001.use1.cache.amazonaws.com:6379

# In Python (redis-py):
import redis

r = redis.Redis(
    host="myapp-redis.abc123.0001.use1.cache.amazonaws.com",
    port=6379,
    password="your-strong-redis-password",
    ssl=True,               # ElastiCache uses TLS
    decode_responses=True
)
```

---

## 6. 📦 ECR — Elastic Container Registry

Bash

```
# ─────────────────────────────────────────
# ECR: AWS's private Docker registry
# Alternative to Docker Hub or GHCR
# Tight integration with ECS and Lambda
# ─────────────────────────────────────────

# Create a repository
aws ecr create-repository \
    --repository-name myapp \
    --image-scanning-configuration scanOnPush=true \
    --encryption-configuration encryptionType=AES256 \
    --region us-east-1

# Get repository URI
REPO_URI=$(aws ecr describe-repositories \
    --repository-names myapp \
    --query "repositories[0].repositoryUri" \
    --output text)
echo $REPO_URI
# 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp

# Login to ECR
aws ecr get-login-password --region us-east-1 | \
    docker login \
    --username AWS \
    --password-stdin \
    123456789012.dkr.ecr.us-east-1.amazonaws.com

# Build, tag, and push
docker build -t myapp:latest .
docker tag myapp:latest $REPO_URI:latest
docker tag myapp:latest $REPO_URI:v1.2.3
docker push $REPO_URI:latest
docker push $REPO_URI:v1.2.3

# List images
aws ecr list-images --repository-name myapp

# Set lifecycle policy (clean up old images)
aws ecr put-lifecycle-policy \
    --repository-name myapp \
    --lifecycle-policy-text '{
        "rules": [
            {
                "rulePriority": 1,
                "description": "Keep last 10 images",
                "selection": {
                    "tagStatus": "any",
                    "countType": "imageCountMoreThan",
                    "countNumber": 10
                },
                "action": {"type": "expire"}
            }
        ]
    }'

# GitHub Actions: login to ECR
# In workflow:
# - name: Login to ECR
#   uses: aws-actions/amazon-ecr-login@v2
#   env:
#     AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
#     AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
#     AWS_REGION: us-east-1
```

---

## 7. 🐳 ECS — Elastic Container Service

### What is ECS?

text

```
ECS = AWS's managed container orchestration.
      Like Kubernetes but simpler.
      Runs your Docker containers at scale.

Two launch types:
  Fargate:  serverless containers (AWS manages servers)
            You only pay for the container resources
            No EC2 management
            Best for: most use cases

  EC2:      you manage the EC2 instances
            More control, potentially cheaper at scale
            Best for: specific instance types needed

Key concepts:
  Cluster:         Group of containers (like a namespace)
  Task Definition: Blueprint for a container (image, CPU, RAM, env vars)
  Task:            A running instance of a Task Definition
  Service:         Keeps N tasks running, handles health/restart
  Container:       The actual running Docker container
```

Python

```
# ecs_demo.py
import boto3
import json

ecs = boto3.client("ecs", region_name="us-east-1")


def create_task_definition() -> dict:
    """Define how our container should run."""

    response = ecs.register_task_definition(
        family="myapp",              # task family name
        networkMode="awsvpc",        # required for Fargate
        requiresCompatibilities=["FARGATE"],

        # CPU and memory (Fargate pricing is based on these)
        cpu="512",                   # 0.5 vCPU
        memory="1024",               # 1 GB RAM

        # IAM role for the task (access S3, RDS, etc.)
        taskRoleArn="arn:aws:iam::123456789012:role/MyAppTaskRole",
        executionRoleArn="arn:aws:iam::123456789012:role/ecsTaskExecutionRole",

        containerDefinitions=[{
            "name": "myapp",
            "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest",

            "cpu": 512,
            "memory": 1024,
            "essential": True,        # if this container stops, stop task

            "portMappings": [{
                "containerPort": 8000,
                "protocol": "tcp"
            }],

            # Environment variables from Secrets Manager (secure!)
            "secrets": [
                {
                    "name": "DATABASE_URL",
                    "valueFrom": (
                        "arn:aws:secretsmanager:us-east-1:123456789012:"
                        "secret:myapp/database-url"
                    )
                },
                {
                    "name": "SECRET_KEY",
                    "valueFrom": (
                        "arn:aws:secretsmanager:us-east-1:123456789012:"
                        "secret:myapp/secret-key"
                    )
                },
            ],

            # Non-secret environment variables
            "environment": [
                {"name": "APP_ENV", "value": "production"},
                {"name": "PORT", "value": "8000"},
                {"name": "WORKERS", "value": "2"},
            ],

            # Logging → CloudWatch
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "/ecs/myapp",
                    "awslogs-region": "us-east-1",
                    "awslogs-stream-prefix": "ecs",
                    "awslogs-create-group": "true"
                }
            },

            # Health check
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "curl -f http://localhost:8000/health || exit 1"
                ],
                "interval": 30,
                "timeout": 5,
                "retries": 3,
                "startPeriod": 60
            },
        }],

        tags=[
            {"key": "Project", "value": "myapp"},
            {"key": "Environment", "value": "production"},
        ]
    )

    task_def = response["taskDefinition"]
    print(f"Task definition: {task_def['taskDefinitionArn']}")
    return task_def


def create_ecs_service(
    cluster_name: str,
    task_def_arn: str,
    target_group_arn: str,
    subnet_ids: list,
    security_group_id: str,
) -> dict:
    """Create an ECS service that keeps tasks running."""

    response = ecs.create_service(
        cluster=cluster_name,
        serviceName="myapp",
        taskDefinition=task_def_arn,
        desiredCount=2,              # run 2 containers

        launchType="FARGATE",

        # Networking
        networkConfiguration={
            "awsvpcConfiguration": {
                "subnets": subnet_ids,          # private subnets
                "securityGroups": [security_group_id],
                "assignPublicIp": "DISABLED",   # private network only!
            }
        },

        # Connect to load balancer
        loadBalancers=[{
            "targetGroupArn": target_group_arn,
            "containerName": "myapp",
            "containerPort": 8000,
        }],

        # Deployment configuration
        deploymentConfiguration={
            "minimumHealthyPercent": 100,   # keep 100% healthy during deploy
            "maximumPercent": 200,          # allow 200% during rolling update
            "deploymentCircuitBreaker": {
                "enable": True,
                "rollback": True,           # auto-rollback on failure!
            }
        },

        # Health check grace period (seconds)
        healthCheckGracePeriodSeconds=60,

        # Auto-scaling (handled separately with Application Auto Scaling)
        enableExecuteCommand=True,   # allow `aws ecs execute-command`

        tags=[
            {"key": "Project", "value": "myapp"},
        ]
    )

    service = response["service"]
    print(f"Created ECS service: {service['serviceArn']}")
    return service


def update_service_image(
    cluster: str,
    service_name: str,
    new_image: str
) -> None:
    """Deploy a new image to ECS (rolling update)."""

    # Get current task definition
    response = ecs.describe_services(
        cluster=cluster,
        services=[service_name]
    )
    task_def_arn = response["services"][0]["taskDefinition"]

    # Get task definition
    task_def = ecs.describe_task_definition(
        taskDefinition=task_def_arn
    )["taskDefinition"]

    # Update image in container definition
    containers = task_def["containerDefinitions"]
    for container in containers:
        if container["name"] == "myapp":
            container["image"] = new_image

    # Register new task definition
    new_task_def = ecs.register_task_definition(
        family=task_def["family"],
        networkMode=task_def["networkMode"],
        requiresCompatibilities=task_def["requiresCompatibilities"],
        cpu=task_def["cpu"],
        memory=task_def["memory"],
        taskRoleArn=task_def["taskRoleArn"],
        executionRoleArn=task_def["executionRoleArn"],
        containerDefinitions=containers,
    )

    new_task_def_arn = new_task_def["taskDefinition"]["taskDefinitionArn"]

    # Update service with new task definition
    ecs.update_service(
        cluster=cluster,
        service=service_name,
        taskDefinition=new_task_def_arn,
        forceNewDeployment=True
    )

    print(f"Deploying new image: {new_image}")
    print(f"Task definition: {new_task_def_arn}")


# Deploy from GitHub Actions:
# aws ecs update-service \
#     --cluster myapp-cluster \
#     --service myapp \
#     --task-definition myapp:NEW_VERSION \
#     --force-new-deployment
```

---

## 8. ⚡ Lambda — Serverless Functions

Python

```
# lambda_demo.py
"""
Lambda = run code without managing servers.
Pay per invocation (often free tier covers everything).

Use cases for backends:
  ✅ Webhooks (process Stripe events)
  ✅ Image processing (resize on upload)
  ✅ Scheduled jobs (reports, cleanup)
  ✅ API backends (simple APIs)
  ✅ Email sending triggers
  ✅ Data pipeline processing

NOT great for:
  ❌ Long-running processes (max 15 min)
  ❌ Persistent connections (WebSockets)
  ❌ High-memory workloads
"""
import boto3
import json
import zipfile
import io

lambda_client = boto3.client("lambda", region_name="us-east-1")


def create_lambda_function(function_name: str) -> dict:
    """Create a Lambda function from a zip file."""

    # Package the function code
    zip_buffer = io.BytesIO()
    with zipfile.ZipFile(zip_buffer, "w") as zf:
        zf.writestr("lambda_function.py", """
import json
import boto3
from datetime import datetime

def handler(event, context):
    '''
    Lambda handler function.
    event = the trigger data (HTTP request, S3 event, etc.)
    context = runtime info
    '''
    print(f"Event: {json.dumps(event)}")

    # Process the event
    if event.get("source") == "aws.events":
        # Scheduled job (CloudWatch Events)
        return cleanup_old_data()

    # HTTP request (API Gateway)
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({
            "message": "Lambda response!",
            "timestamp": datetime.now().isoformat()
        })
    }

def cleanup_old_data():
    # Example: delete expired sessions from DynamoDB
    dynamodb = boto3.resource("dynamodb")
    table = dynamodb.Table("sessions")
    # ... cleanup logic
    return {"cleaned": True}
""")
    zip_buffer.seek(0)

    response = lambda_client.create_function(
        FunctionName=function_name,
        Runtime="python3.12",
        Role="arn:aws:iam::123456789012:role/lambda-execution-role",
        Handler="lambda_function.handler",
        Code={"ZipFile": zip_buffer.read()},

        # Resources
        MemorySize=256,           # MB (128-10240)
        Timeout=30,               # seconds (max 900 = 15 min)

        # Environment variables
        Environment={
            "Variables": {
                "APP_ENV": "production",
                "TABLE_NAME": "sessions",
            }
        },

        # Logging
        Layers=[],                # optional: Lambda layers for deps

        Tags={
            "Project": "myapp",
            "Environment": "production",
        }
    )

    print(f"Lambda created: {response['FunctionArn']}")
    return response


def add_s3_trigger(
    function_name: str,
    bucket_name: str,
    prefix: str = "uploads/",
) -> None:
    """Trigger Lambda when file is uploaded to S3."""

    # Add S3 permission to invoke Lambda
    lambda_client.add_permission(
        FunctionName=function_name,
        StatementId="AllowS3Invoke",
        Action="lambda:InvokeFunction",
        Principal="s3.amazonaws.com",
        SourceArn=f"arn:aws:s3:::{bucket_name}",
    )

    # Configure S3 to trigger Lambda
    s3 = boto3.client("s3")
    s3.put_bucket_notification_configuration(
        Bucket=bucket_name,
        NotificationConfiguration={
            "LambdaFunctionConfigurations": [{
                "LambdaFunctionArn": (
                    f"arn:aws:lambda:us-east-1:123456789012:"
                    f"function:{function_name}"
                ),
                "Events": ["s3:ObjectCreated:*"],
                "Filter": {
                    "Key": {
                        "FilterRules": [{
                            "Name": "prefix",
                            "Value": prefix
                        }]
                    }
                }
            }]
        }
    )
    print(f"S3 trigger added: {bucket_name}/{prefix}")


# Add scheduled trigger (like a cron job)
def add_scheduled_trigger(
    function_name: str,
    schedule: str = "rate(1 day)",   # or "cron(0 2 * * ? *)"
) -> None:
    """Run Lambda on a schedule."""
    events = boto3.client("events")

    # Create CloudWatch Events rule
    rule = events.put_rule(
        Name=f"{function_name}-schedule",
        ScheduleExpression=schedule,
        State="ENABLED",
    )

    # Add Lambda as target
    events.put_targets(
        Rule=f"{function_name}-schedule",
        Targets=[{
            "Id": "1",
            "Arn": (
                f"arn:aws:lambda:us-east-1:123456789012:"
                f"function:{function_name}"
            ),
        }]
    )

    # Allow Events to invoke Lambda
    lambda_client.add_permission(
        FunctionName=function_name,
        StatementId="AllowCloudWatchEvents",
        Action="lambda:InvokeFunction",
        Principal="events.amazonaws.com",
        SourceArn=rule["RuleArn"],
    )
    print(f"Scheduled trigger added: {schedule}")
```

---

## 9. 📊 CloudWatch — Monitoring & Logging

Python

```
# cloudwatch_demo.py
import boto3
from datetime import datetime, timedelta

cloudwatch = boto3.client("cloudwatch", region_name="us-east-1")
logs = boto3.client("logs", region_name="us-east-1")


# ─────────────────────────────────────────
# LOGGING
# ─────────────────────────────────────────
def setup_log_group(log_group: str, retention_days: int = 30) -> None:
    """Create a CloudWatch log group."""
    try:
        logs.create_log_group(logGroupName=log_group)
        logs.put_retention_policy(
            logGroupName=log_group,
            retentionInDays=retention_days
        )
        print(f"Log group created: {log_group}")
    except logs.exceptions.ResourceAlreadyExistsException:
        print(f"Log group exists: {log_group}")


# Your app sends logs via stdout → ECS → CloudWatch automatically
# For manual log sending:
def send_log(
    log_group: str,
    log_stream: str,
    message: str,
) -> None:
    """Send a log message to CloudWatch."""
    try:
        # Create stream if needed
        try:
            logs.create_log_stream(
                logGroupName=log_group,
                logStreamName=log_stream
            )
        except logs.exceptions.ResourceAlreadyExistsException:
            pass

        # Send log event
        logs.put_log_events(
            logGroupName=log_group,
            logStreamName=log_stream,
            logEvents=[{
                "timestamp": int(datetime.now().timestamp() * 1000),
                "message": message
            }]
        )
    except Exception as e:
        print(f"CloudWatch log error: {e}")


# ─────────────────────────────────────────
# METRICS
# ─────────────────────────────────────────
def put_custom_metric(
    namespace: str,
    metric_name: str,
    value: float,
    unit: str = "Count",
    dimensions: list = None,
) -> None:
    """Send a custom metric to CloudWatch."""

    metric_data = {
        "MetricName": metric_name,
        "Value": value,
        "Unit": unit,
        "Timestamp": datetime.now(),
    }

    if dimensions:
        metric_data["Dimensions"] = dimensions

    cloudwatch.put_metric_data(
        Namespace=namespace,
        MetricData=[metric_data]
    )


# In your FastAPI app middleware:
def track_request_metrics(endpoint: str, status: int, duration_ms: float):
    """Track API metrics in CloudWatch."""

    # Request count
    put_custom_metric(
        namespace="MyApp",
        metric_name="RequestCount",
        value=1,
        unit="Count",
        dimensions=[
            {"Name": "Endpoint", "Value": endpoint},
            {"Name": "StatusCode", "Value": str(status)},
        ]
    )

    # Response time
    put_custom_metric(
        namespace="MyApp",
        metric_name="ResponseTime",
        value=duration_ms,
        unit="Milliseconds",
        dimensions=[{"Name": "Endpoint", "Value": endpoint}]
    )

    # Error tracking
    if status >= 500:
        put_custom_metric(
            namespace="MyApp",
            metric_name="ErrorCount",
            value=1,
            unit="Count",
        )


# ─────────────────────────────────────────
# ALARMS
# ─────────────────────────────────────────
def create_alarm(
    alarm_name: str,
    metric_name: str,
    threshold: float,
    comparison: str = "GreaterThanThreshold",
    sns_topic_arn: str = None,
) -> None:
    """Create a CloudWatch alarm."""

    actions = []
    if sns_topic_arn:
        actions = [sns_topic_arn]   # send notification via SNS

    cloudwatch.put_metric_alarm(
        AlarmName=alarm_name,
        MetricName=metric_name,
        Namespace="MyApp",
        Statistic="Sum",
        Period=300,             # 5 minutes
        EvaluationPeriods=2,    # 2 consecutive periods
        Threshold=threshold,
        ComparisonOperator=comparison,
        AlarmActions=actions,
        OKActions=actions,      # notify when back to OK
        TreatMissingData="notBreaching",
    )
    print(f"Alarm created: {alarm_name}")


# Practical alarms:
def setup_production_alarms(sns_arn: str) -> None:
    """Set up alarms for production monitoring."""

    # High error rate
    create_alarm(
        "HighErrorRate",
        "ErrorCount",
        threshold=10,           # 10 errors in 5 min = alarm
        sns_topic_arn=sns_arn
    )

    # High response time
    cloudwatch.put_metric_alarm(
        AlarmName="HighResponseTime",
        MetricName="ResponseTime",
        Namespace="MyApp",
        Statistic="p95",        # 95th percentile
        Period=300,
        EvaluationPeriods=3,
        Threshold=500,          # 500ms P95 = alarm
        ComparisonOperator="GreaterThanThreshold",
        AlarmActions=[sns_arn],
    )

    # RDS CPU high
    cloudwatch.put_metric_alarm(
        AlarmName="RDS-HighCPU",
        MetricName="CPUUtilization",
        Namespace="AWS/RDS",
        Dimensions=[{
            "Name": "DBInstanceIdentifier",
            "Value": "myapp-postgres"
        }],
        Statistic="Average",
        Period=300,
        EvaluationPeriods=3,
        Threshold=80,           # 80% CPU
        ComparisonOperator="GreaterThanThreshold",
        AlarmActions=[sns_arn],
    )
```

---

## 10. 🌐 Route 53 & ALB

Python

```
# route53_alb_demo.py
import boto3

route53 = boto3.client("route53")
elbv2 = boto3.client("elbv2", region_name="us-east-1")


def create_load_balancer(
    name: str,
    subnet_ids: list,
    security_group_id: str,
) -> dict:
    """Create an Application Load Balancer."""

    # Create ALB
    alb = elbv2.create_load_balancer(
        Name=name,
        Subnets=subnet_ids,                  # public subnets
        SecurityGroups=[security_group_id],
        Scheme="internet-facing",            # public ALB
        Type="application",
        IpAddressType="ipv4",
        Tags=[
            {"Key": "Name", "Value": name},
            {"Key": "Environment", "Value": "production"},
        ]
    )["LoadBalancers"][0]

    alb_arn = alb["LoadBalancerArn"]
    alb_dns = alb["DNSName"]

    print(f"ALB created: {alb_dns}")

    # Create target group (points to ECS tasks)
    tg = elbv2.create_target_group(
        Name=f"{name}-tg",
        Protocol="HTTP",
        Port=8000,
        VpcId="vpc-your-vpc-id",
        TargetType="ip",                     # required for Fargate
        HealthCheckProtocol="HTTP",
        HealthCheckPath="/health",
        HealthCheckIntervalSeconds=30,
        HealthCheckTimeoutSeconds=5,
        HealthyThresholdCount=2,
        UnhealthyThresholdCount=3,
    )["TargetGroups"][0]

    tg_arn = tg["TargetGroupArn"]

    # Create HTTPS listener
    elbv2.create_listener(
        LoadBalancerArn=alb_arn,
        Protocol="HTTPS",
        Port=443,
        SslPolicy="ELBSecurityPolicy-TLS13-1-2-2021-06",
        Certificates=[{
            "CertificateArn": (
                "arn:aws:acm:us-east-1:123456789012:certificate/abc-123"
            )
        }],
        DefaultActions=[{
            "Type": "forward",
            "TargetGroupArn": tg_arn,
        }]
    )

    # HTTP listener → redirect to HTTPS
    elbv2.create_listener(
        LoadBalancerArn=alb_arn,
        Protocol="HTTP",
        Port=80,
        DefaultActions=[{
            "Type": "redirect",
            "RedirectConfig": {
                "Protocol": "HTTPS",
                "Port": "443",
                "StatusCode": "HTTP_301",
            }
        }]
    )

    return {"alb_dns": alb_dns, "tg_arn": tg_arn}


def create_dns_record(
    hosted_zone_id: str,
    domain: str,
    alb_dns: str,
    alb_hosted_zone_id: str,
) -> None:
    """Point your domain to the load balancer via Route 53."""

    route53.change_resource_record_sets(
        HostedZoneId=hosted_zone_id,
        ChangeBatch={
            "Changes": [
                {
                    "Action": "UPSERT",
                    "ResourceRecordSet": {
                        "Name": domain,
                        "Type": "A",
                        # Alias record: free, auto-updates if ALB IP changes
                        "AliasTarget": {
                            "HostedZoneId": alb_hosted_zone_id,
                            "DNSName": alb_dns,
                            "EvaluateTargetHealth": True,
                        }
                    }
                },
                # Also create www subdomain
                {
                    "Action": "UPSERT",
                    "ResourceRecordSet": {
                        "Name": f"www.{domain}",
                        "Type": "A",
                        "AliasTarget": {
                            "HostedZoneId": alb_hosted_zone_id,
                            "DNSName": alb_dns,
                            "EvaluateTargetHealth": True,
                        }
                    }
                }
            ]
        }
    )
    print(f"DNS record created: {domain} → {alb_dns}")
```

---

## 11. 🔐 Secrets Manager

Python

```
# secrets_manager_demo.py
import boto3
import json

sm = boto3.client("secretsmanager", region_name="us-east-1")


def store_secret(name: str, value: dict) -> str:
    """Store a secret in AWS Secrets Manager."""
    try:
        response = sm.create_secret(
            Name=name,
            Description=f"Secret for {name}",
            SecretString=json.dumps(value),
            Tags=[{"Key": "Project", "Value": "myapp"}]
        )
    except sm.exceptions.ResourceExistsException:
        # Update existing secret
        response = sm.put_secret_value(
            SecretId=name,
            SecretString=json.dumps(value)
        )

    print(f"Secret stored: {name}")
    return response["ARN"]


def get_secret(name: str) -> dict:
    """Retrieve a secret from Secrets Manager."""
    response = sm.get_secret_value(SecretId=name)
    return json.loads(response["SecretString"])


# Store all app secrets:
def setup_production_secrets():
    # Database credentials
    store_secret("myapp/database", {
        "url": "postgresql://user:pass@rds-endpoint.amazonaws.com:5432/mydb",
        "username": "myapp_admin",
        "password": "super-secure-password",
        "host": "myapp-postgres.abc123.us-east-1.rds.amazonaws.com",
        "port": 5432,
        "dbname": "myapp_db",
    })

    # App secrets
    store_secret("myapp/app", {
        "secret_key": "your-32-char-secret-key-here!!",
        "jwt_secret": "your-32-char-jwt-secret-here!!",
    })

    # External API keys
    store_secret("myapp/stripe", {
        "secret_key": "sk_live_...",
        "webhook_secret": "whsec_...",
    })


# In your app (runs on ECS with proper IAM role):
class ProductionSettings:
    def __init__(self):
        secrets = get_secret("myapp/app")
        self.SECRET_KEY = secrets["secret_key"]
        self.JWT_SECRET_KEY = secrets["jwt_secret"]

        db_secrets = get_secret("myapp/database")
        self.DATABASE_URL = db_secrets["url"]
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                    AWS ARCHITECTURE                            │
│                                                                │
│  Internet → Route 53 → ALB → ECS (Fargate containers)        │
│                                   ↓        ↓        ↓         │
│                               RDS       ElastiCache  S3        │
│                            (PostgreSQL)  (Redis)  (Files)      │
│                                                                │
│  SECURITY LAYERS:                                              │
│  Public subnet:   ALB, NAT Gateway                            │
│  Private subnet:  ECS tasks, RDS, ElastiCache                 │
│  Security groups: strict inbound rules                        │
│  IAM roles:       no hardcoded credentials                    │
│  Secrets Manager: encrypted secret storage                    │
│                                                                │
│  KEY SERVICES:                                                 │
│  IAM         → who can do what (foundation)                   │
│  EC2         → virtual machines                               │
│  RDS         → managed PostgreSQL (auto backup, failover)     │
│  S3          → object storage (files, backups)                │
│  ElastiCache → managed Redis                                  │
│  ECR         → Docker image registry                          │
│  ECS Fargate → serverless container hosting                   │
│  Lambda      → serverless functions (webhooks, jobs)          │
│  CloudWatch  → logs, metrics, alarms                         │
│  Route 53    → DNS management                                 │
│  ALB         → load balancer, HTTPS termination              │
│  Secrets Mgr → encrypted secrets storage                     │
│                                                                │
│  FREE TIER (12 months):                                        │
│  EC2:        750 hours t2.micro/month                        │
│  RDS:        750 hours db.t2.micro/month                     │
│  S3:         5GB storage                                      │
│  Lambda:     1M requests/month (always free)                 │
│  CloudWatch: 10 metrics, 5GB logs                            │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is the principle of least privilege in IAM?
2.  What is an IAM role vs IAM user? When do you use each?
3.  Why should you use IAM roles instead of access keys on EC2?
4.  What is an EC2 security group?
5.  Why use RDS instead of PostgreSQL on EC2?
6.  What is Multi-AZ in RDS and why enable it?
7.  What is S3 and what do you use it for in a backend?
8.  What is a presigned URL and when do you need one?
9.  What is the difference between ECS and EC2?
10. What is Fargate? How does it differ from ECS on EC2?
11. What is Lambda? What are its limitations?
12. What does CloudWatch do? What are alarms?
13. What is Secrets Manager and why use it?
14. What is an ALB and what does it do?
15. What is Route 53?
```

---

## 🛠️ Practice Exercises

Bash

```
# Exercise 1: AWS Free Tier Setup
# 1. Create AWS account
# 2. Create IAM admin user (don't use root!)
# 3. Set up MFA on root and admin
# 4. Configure AWS CLI with admin credentials
# 5. Set up billing alarm: alert at $10/month
# 6. Verify: aws sts get-caller-identity

# Exercise 2: EC2 + Deploy Your App
# 1. Launch t2.micro Ubuntu instance
# 2. Create security group (SSH + HTTP + HTTPS)
# 3. SSH into instance
# 4. Install Docker, clone your repo
# 5. Run your FastAPI app in Docker
# 6. Access via public IP
# 7. Allocate and assign Elastic IP

# Exercise 3: S3 File Storage
# In your FastAPI app, add:
# - Upload endpoint: POST /upload → stores in S3
# - Generates presigned URL for access
# - Direct upload: POST /upload/direct → returns presigned POST
# - Delete endpoint: DELETE /files/{key}
# Test all three scenarios

# Exercise 4: RDS PostgreSQL
# 1. Create RDS t2.micro instance
# 2. Connect via SSH tunnel through EC2
# 3. Update your FastAPI app to use RDS URL
# 4. Run migrations against RDS
# 5. Test data persists through app restart
# 6. Take a manual snapshot

# Exercise 5: ECS Fargate Deployment
# 1. Create ECR repository
# 2. Push your Docker image
# 3. Create ECS cluster (Fargate)
# 4. Create task definition (your app + CloudWatch logging)
# 5. Create ECS service (2 tasks)
# 6. Create ALB and target group
# 7. Connect ECS to ALB
# 8. Verify app accessible via ALB DNS
# 9. Update image and watch rolling deploy
```

---

## ✅ Phase 7.4 Complete!

**You now know:**

text

```
✅ IAM — users, roles, policies, least privilege
✅ EC2 — instances, security groups, user data
✅ RDS — managed PostgreSQL, Multi-AZ, backups
✅ S3 — buckets, uploads, presigned URLs, direct upload
✅ ElastiCache — managed Redis setup
✅ ECR — Docker image registry
✅ ECS Fargate — serverless containers, task definitions, services
✅ Lambda — serverless functions, triggers, scheduling
✅ CloudWatch — logs, metrics, custom alarms
✅ Route 53 — DNS, alias records
✅ ALB — load balancer, HTTPS termination
✅ Secrets Manager — encrypted secrets storage
✅ boto3 — AWS Python SDK
✅ Complete production architecture on AWS
```