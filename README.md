# Production-Ready 3-Tier Web App Infrastructure (CloudFormation)

This project provisions a complete production-grade 3-tier architecture on AWS using **AWS CloudFormation**. It includes modular stacks for networking, compute, storage, database, and security — designed for real-world scalability and maintainability.

---

## 🔧 Stacks

### ☁️ Networking

* **VPC**: Custom VPC with CIDR 10.0.0.0/16
* **Subnets**: 2 Public + 2 Private Subnets across multiple Availability Zones
* **Internet Gateway**: Enables public subnet connectivity
* **NAT Gateway**: Enables private subnets to access the internet securely
* **Route Tables**: Configured for both public and private subnet routing

### 🛡️ Security

* **ALBSG**: Allows HTTP from internet to ALB
* **InstanceSG**: Allows HTTP from ALB to EC2
* **RDSSG**: Allows MySQL access from EC2 to RDS (port 3306)

### 🖥️ Compute

* **Launch Template**: Amazon Linux 2 + Apache install via User Data
* **Auto Scaling Group (ASG)**: Launches EC2 instances in public subnets
* **Application Load Balancer (ALB)**: Distributes HTTP traffic to instances

### 🛢️ Database

* **RDS Instance**: MySQL 8.0 in private subnets
* **DB Subnet Group**: Includes 2 private subnets

### 🗃️ Storage

* **S3 Bucket**: With versioning and public access blocked

---

## ✅ Features

* Modular YAML templates with independent stack deployment
* Uses `!ImportValue` for clean decoupling between stacks
* Production-grade network layout (isolated tiers)
* Web server auto-deploys Apache + test HTML
* Security groups follow least-privilege principles
* All sensitive params (e.g., DB password) are passed securely

---

## 📁 Directory Structure

```
cloudformation-3tier-app/
├── compute/
│   └── ec2-asg-alb.yaml
├── database/
│   └── rds.yaml
├── networking/
│   ├── vpc.yaml
│   ├── igw-routing.yaml
│   └── nat-routing.yaml
├── security/
│   └── security-groups.yaml
├── storage/
│   └── s3.yaml
├── parent(optional).yaml
└── .gitignore
```

---

## 🚀 Deployment Options

### 1️⃣ Manual Deployment (Stack-by-Stack) – PowerShell

```powershell
# 1. Create core networking
aws cloudformation deploy `
  --template-file networking/vpc.yaml `
  --stack-name my-vpc-stack `
  --region eu-west-2

# 2. Internet Gateway & Routing
aws cloudformation deploy `
  --template-file networking/igw-routing.yaml `
  --stack-name routing-stack `
  --parameter-overrides VPCId=... PublicSubnet1Id=... PublicSubnet2Id=... InternetGatewayId=... `
  --region eu-west-2

# 3. NAT Gateway & Private Routing
aws cloudformation deploy `
  --template-file networking/nat-routing.yaml `
  --stack-name nat-stack `
  --parameter-overrides VPCId=... PublicSubnet1Id=... PrivateSubnet1Id=... PrivateSubnet2Id=... `
  --region eu-west-2

# 4. Security Groups
aws cloudformation deploy `
  --template-file security/security-groups.yaml `
  --stack-name security-stack `
  --parameter-overrides VPCId=... `
  --region eu-west-2

# 5. Compute (ALB, ASG, EC2)
aws cloudformation deploy `
  --template-file compute/ec2-asg-alb.yaml `
  --stack-name compute-stack `
  --parameter-overrides VPCId=... PublicSubnet1Id=... PublicSubnet2Id=... `
  --region eu-west-2

# 6. Database (RDS)
aws cloudformation deploy `
  --template-file database/rds.yaml `
  --stack-name rds-stack `
  --parameter-overrides PrivateSubnet1Id=... PrivateSubnet2Id=... DBPassword=... `
  --region eu-west-2

# 7. Storage (S3)
aws cloudformation deploy `
  --template-file storage/s3.yaml `
  --stack-name s3-stack `
  --parameter-overrides BucketName=your-unique-bucket-name `
  --region eu-west-2
```

---

### 2️⃣ One-Click Deploy (Optional Nested Stack)

This project includes an optional `parent(optional).yaml` file to deploy the entire 3-tier infrastructure using nested stacks.

#### 📦 How to Use:

1. **Upload all templates** to an S3 bucket (e.g., `my-3tier-templates`) with the same folder structure:
   ```
   s3://my-3tier-templates/networking/vpc.yaml
   s3://my-3tier-templates/networking/igw-routing.yaml
   s3://my-3tier-templates/networking/nat-routing.yaml
   s3://my-3tier-templates/security/security-groups.yaml
   s3://my-3tier-templates/compute/ec2-asg-alb.yaml
   s3://my-3tier-templates/database/rds.yaml
   s3://my-3tier-templates/storage/s3.yaml
   ```

2. **Deploy the nested parent stack:**
   ```powershell
   aws cloudformation create-stack `
     --stack-name 3tier-nested-app `
     --template-body file://parent(optional).yaml `
     --parameters ParameterKey=TemplateBucket,ParameterValue=my-3tier-templates `
     --capabilities CAPABILITY_NAMED_IAM
   ```

> This is optional and included for convenience. The project remains modular and production-ready with or without nested deployment.

---

## 🔒 Security Notes

* DB password is passed via `--parameter-overrides` and is not stored in plaintext.
* Public access is blocked on the S3 bucket.
* EC2 cannot access RDS unless whitelisted via Security Group.

---

## 👤 Author

**Rami Alshaar**  
[GitHub Profile](https://github.com/Rami-shaar)
