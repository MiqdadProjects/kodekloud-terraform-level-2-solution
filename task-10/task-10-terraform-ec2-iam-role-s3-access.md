# 🌟 Task 10 - Provision EC2 with IAM Role for Secure S3 Log Upload Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** wants EC2 instances to **securely upload application logs to S3** using **IAM roles** — the AWS-recommended approach for granting EC2 instances access to other AWS services **without embedding credentials**. By attaching an IAM role with a scoped S3 `PutObject` policy, the EC2 instance can securely write logs to the designated bucket using instance metadata-based temporary credentials.

**Requirements:**
- Create an **EC2 instance** named **`datacenter-ec2`** using the latest Amazon Linux 2 AMI (fetched dynamically)
- Create an **S3 bucket** named **`datacenter-logs-2121`** for log storage
- Create an **IAM role** named **`datacenter-role`** with an EC2 trust policy
- Create an **IAM policy** named **`datacenter-access-policy`** granting `s3:PutObject` on the logs bucket
- Attach the IAM role to the EC2 instance via an **instance profile**
- All resources in a single **`main.tf`** file
- Use **`variables.tf`** to declare: `KKE_BUCKET_NAME`, `KKE_POLICY_NAME`, `KKE_ROLE_NAME`
- Use **`terraform.tfvars`** to assign values to all variables
- Use **`data.tf`** to dynamically fetch the latest Amazon Linux 2 AMI
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Build a complete, secure AWS log ingestion stack — EC2 instance with a purpose-scoped IAM role that grants minimal S3 write access — without storing any credentials on the instance.

💡 **Note:** IAM instance profiles are the bridge between IAM roles and EC2 instances. Roles cannot be directly attached to EC2 — they must be wrapped in an instance profile (`aws_iam_instance_profile`). The instance then retrieves short-lived credentials via the EC2 Metadata Service (IMDS) automatically.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- S3 bucket (`datacenter-logs-2121`) — destination for EC2 log uploads
- IAM role (`datacenter-role`) — EC2 trust relationship
- IAM policy (`datacenter-access-policy`) — `s3:PutObject` on the logs bucket
- IAM policy attachment — links policy to role
- IAM instance profile — wraps the role for EC2 attachment
- EC2 instance (`datacenter-ec2`) — Amazon Linux 2, `t2.micro`, with instance profile

**Working Directory:** `/home/bob/terraform`

**IAM Permission Chain:**
```
datacenter-ec2
    │  attached via instance profile
    ▼
datacenter-role  (Trust: ec2.amazonaws.com → sts:AssumeRole)
    │  policy attached
    ▼
datacenter-access-policy
    │  grants
    ▼
s3:PutObject on arn:aws:s3:::datacenter-logs-2121/*
```

**Resource Summary:**
| Resource | Terraform Type | Name |
|----------|---------------|------|
| S3 Bucket | `aws_s3_bucket` | `datacenter-logs-2121` |
| IAM Role | `aws_iam_role` | `datacenter-role` |
| IAM Policy | `aws_iam_policy` | `datacenter-access-policy` |
| Policy Attachment | `aws_iam_role_policy_attachment` | — |
| Instance Profile | `aws_iam_instance_profile` | `datacenter-instance-profile` |
| EC2 Instance | `aws_instance` | `datacenter-ec2` |
| AMI Data Source | `data.aws_ami` | `amazon_linux_2` (dynamic) |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`data.aws_ami`** (`data.tf`): Dynamically resolves the latest Amazon Linux 2 AMI — no hardcoded AMI IDs
- **`aws_s3_bucket`**: Creates the log destination bucket
- **`aws_iam_role`**: Creates an IAM role with an EC2 trust policy (`sts:AssumeRole`)
- **`aws_iam_policy`**: Creates a minimal policy permitting only `s3:PutObject` on the bucket ARN
- **`aws_iam_role_policy_attachment`**: Attaches the policy to the role
- **`aws_iam_instance_profile`**: Wraps the role so it can be attached to EC2
- **`aws_instance`**: EC2 carrying the instance profile — gains S3 write access via IMDS credentials

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # All AWS resources (S3, IAM role, policy, EC2)
├── data.tf            # AMI data source (Amazon Linux 2)
├── variables.tf       # Variable declarations (no defaults)
├── terraform.tfvars   # Variable values
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

---

### Step 2: Create Variables Definition File

Create `variables.tf` with the three required variable declarations:

```bash
cat > variables.tf << 'EOF'
variable "KKE_BUCKET_NAME" {
  type = string
}

variable "KKE_POLICY_NAME" {
  type = string
}

variable "KKE_ROLE_NAME" {
  type = string
}
EOF
```

**Configuration Breakdown:**
- Three string variables — no defaults, all values supplied by `terraform.tfvars`
- No `description` field is required by the task, but it's a best practice to add one

**File Content (`variables.tf`):**
```hcl
variable "KKE_BUCKET_NAME" {
  type = string
}

variable "KKE_POLICY_NAME" {
  type = string
}

variable "KKE_ROLE_NAME" {
  type = string
}
```

---

### Step 3: Create Terraform Variables File

Create `terraform.tfvars` with actual values for all three variables:

```bash
cat > terraform.tfvars << 'EOF'
KKE_BUCKET_NAME = "datacenter-logs-2121"
KKE_POLICY_NAME = "datacenter-access-policy"
KKE_ROLE_NAME   = "datacenter-role"
EOF
```

**File Content (`terraform.tfvars`):**
```hcl
KKE_BUCKET_NAME = "datacenter-logs-2121"
KKE_POLICY_NAME = "datacenter-access-policy"
KKE_ROLE_NAME   = "datacenter-role"
```

---

### Step 4: Create Data Source File

Create `data.tf` with the Amazon Linux 2 AMI data source:

```bash
cat > data.tf << 'EOF'
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
EOF
```

**Purpose:** Dynamically resolve the latest Amazon Linux 2 AMI ID at plan/apply time — eliminating hardcoded AMI IDs that become stale as Amazon publishes new images.

**Configuration Breakdown:**
- `most_recent = true` — Returns the newest matching AMI when multiple results match
- `owners = ["amazon"]` — Restricts results to official AWS-published Amazon Linux AMIs only (excludes marketplace or community AMIs)
- `filter name = "amzn2-ami-hvm-*-x86_64-gp2"` — HVM virtualization, x86_64 architecture, GP2 EBS root volume — the standard Amazon Linux 2 image type

**File Content (`data.tf`):**
```hcl
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
```

---

### Step 5: Create Main Configuration File

Create `main.tf` with all six resources — S3, IAM role, policy, attachment, instance profile, and EC2:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# S3 Bucket
# ---------------------------
resource "aws_s3_bucket" "logs_bucket" {
  bucket = var.KKE_BUCKET_NAME
}

# ---------------------------
# IAM Role
# ---------------------------
resource "aws_iam_role" "datacenter_role" {
  name = var.KKE_ROLE_NAME

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })
}

# ---------------------------
# IAM Policy
# ---------------------------
resource "aws_iam_policy" "datacenter_policy" {
  name = var.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:PutObject"
        ]
        Resource = "${aws_s3_bucket.logs_bucket.arn}/*"
      }
    ]
  })
}

# Attach policy to role
resource "aws_iam_role_policy_attachment" "attach_policy" {
  role       = aws_iam_role.datacenter_role.name
  policy_arn = aws_iam_policy.datacenter_policy.arn
}

# ---------------------------
# Instance Profile
# ---------------------------
resource "aws_iam_instance_profile" "datacenter_profile" {
  name = "datacenter-instance-profile"
  role = aws_iam_role.datacenter_role.name
}

# ---------------------------
# EC2 Instance
# ---------------------------
resource "aws_instance" "datacenter_ec2" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t2.micro"

  iam_instance_profile = aws_iam_instance_profile.datacenter_profile.name

  tags = {
    Name = "datacenter-ec2"
  }
}
EOF
```

**Configuration Breakdown — Resource by Resource:**

**1. S3 Bucket (`aws_s3_bucket.logs_bucket`):**
- `bucket = var.KKE_BUCKET_NAME` — Named from variable (`datacenter-logs-2121`)
- Acts as the `Resource` target in the IAM policy

**2. IAM Role (`aws_iam_role.datacenter_role`):**
- `name = var.KKE_ROLE_NAME` — Named from variable (`datacenter-role`)
- `assume_role_policy` — Trust policy written as inline JSON using `jsonencode()`:
  - `Principal.Service = "ec2.amazonaws.com"` — Only EC2 instances can assume this role
  - `Action = "sts:AssumeRole"` — The standard permission for assuming a role

**3. IAM Policy (`aws_iam_policy.datacenter_policy`):**
- `name = var.KKE_POLICY_NAME` — Named from variable (`datacenter-access-policy`)
- `policy` — Inline JSON using `jsonencode()`:
  - `Action = "s3:PutObject"` — Grants write access only (no read, no delete, no list)
  - `Resource = "${aws_s3_bucket.logs_bucket.arn}/*"` — Scoped to objects **within** the bucket only (not the bucket itself)

**4. Policy Attachment (`aws_iam_role_policy_attachment.attach_policy`):**
- `role` — References the IAM role name
- `policy_arn` — References the customer-managed policy ARN
- Creates the link: role → policy

**5. Instance Profile (`aws_iam_instance_profile.datacenter_profile`):**
- `name = "datacenter-instance-profile"` — EC2 uses profiles, not roles directly
- `role` — Wraps the IAM role so it can be attached to an EC2 instance

**6. EC2 Instance (`aws_instance.datacenter_ec2`):**
- `ami = data.aws_ami.amazon_linux_2.id` — Dynamic AMI from `data.tf`
- `instance_type = "t2.micro"` — Cost-effective general-purpose instance
- `iam_instance_profile = aws_iam_instance_profile.datacenter_profile.name` — Grants the instance S3 write access via the role
- `tags.Name = "datacenter-ec2"` — Required name tag

**File Content (`main.tf`):**
```hcl
# ---------------------------
# S3 Bucket
# ---------------------------
resource "aws_s3_bucket" "logs_bucket" {
  bucket = var.KKE_BUCKET_NAME
}

# ---------------------------
# IAM Role
# ---------------------------
resource "aws_iam_role" "datacenter_role" {
  name = var.KKE_ROLE_NAME

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })
}

# ---------------------------
# IAM Policy
# ---------------------------
resource "aws_iam_policy" "datacenter_policy" {
  name = var.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:PutObject"
        ]
        Resource = "${aws_s3_bucket.logs_bucket.arn}/*"
      }
    ]
  })
}

# Attach policy to role
resource "aws_iam_role_policy_attachment" "attach_policy" {
  role       = aws_iam_role.datacenter_role.name
  policy_arn = aws_iam_policy.datacenter_policy.arn
}

# ---------------------------
# Instance Profile
# ---------------------------
resource "aws_iam_instance_profile" "datacenter_profile" {
  name = "datacenter-instance-profile"
  role = aws_iam_role.datacenter_role.name
}

# ---------------------------
# EC2 Instance
# ---------------------------
resource "aws_instance" "datacenter_ec2" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t2.micro"

  iam_instance_profile = aws_iam_instance_profile.datacenter_profile.name

  tags = {
    Name = "datacenter-ec2"
  }
}
```

---

### Step 6: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Expected Output:**
```
total 24
drwxr-xr-x 2 bob bob 4096 Feb 25 11:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 10:55 ..
-rw-r--r-- 1 bob bob  155 Feb 25 11:00 data.tf
-rw-r--r-- 1 bob bob 1482 Feb 25 11:00 main.tf
-rw-r--r-- 1 bob bob   96 Feb 25 11:00 terraform.tfvars
-rw-r--r-- 1 bob bob  122 Feb 25 11:00 variables.tf
```

**Success Indicators:**
- ✅ All four required files present
- ✅ No `outputs.tf` (none required by this task)

---

### Step 7: Initialize Terraform

```bash
terraform init
```

**Expected Output:**
```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.31.0...
- Installed hashicorp/aws v5.31.0 (signed by HashiCorp)

Terraform has been successfully initialized!
```

---

### Step 8: Validate Configuration

```bash
terraform validate
```

**Expected Output:**
```
Success! The configuration is valid.
```

**What's Validated:**
- ✅ `jsonencode()` blocks for trust and access policies are syntactically valid
- ✅ `data.aws_ami.amazon_linux_2.id` reference in `aws_instance` resolves correctly
- ✅ `aws_s3_bucket.logs_bucket.arn` reference in policy resolves correctly
- ✅ All variable references match declarations in `variables.tf`

---

### Step 9: Format Configuration Files

```bash
terraform fmt
```

**Expected Output:**
```
data.tf
main.tf
terraform.tfvars
variables.tf
```

---

### Step 10: Review Execution Plan

```bash
terraform plan
```

**Expected Output:**
```
data.aws_ami.amazon_linux_2: Reading...
data.aws_ami.amazon_linux_2: Read complete after 1s [id=ami-0abcd1234efgh5678]

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_iam_instance_profile.datacenter_profile will be created
  + resource "aws_iam_instance_profile" "datacenter_profile" {
      + name = "datacenter-instance-profile"
      + role = "datacenter-role"
    }

  # aws_iam_policy.datacenter_policy will be created
  + resource "aws_iam_policy" "datacenter_policy" {
      + name   = "datacenter-access-policy"
      + policy = jsonencode({...})
    }

  # aws_iam_role.datacenter_role will be created
  + resource "aws_iam_role" "datacenter_role" {
      + name               = "datacenter-role"
      + assume_role_policy = jsonencode({...})
    }

  # aws_iam_role_policy_attachment.attach_policy will be created
  + resource "aws_iam_role_policy_attachment" "attach_policy" {
      + policy_arn = (known after apply)
      + role       = "datacenter-role"
    }

  # aws_instance.datacenter_ec2 will be created
  + resource "aws_instance" "datacenter_ec2" {
      + ami                  = "ami-0abcd1234efgh5678"
      + iam_instance_profile = "datacenter-instance-profile"
      + instance_type        = "t2.micro"
      + tags                 = {
          + "Name" = "datacenter-ec2"
        }
    }

  # aws_s3_bucket.logs_bucket will be created
  + resource "aws_s3_bucket" "logs_bucket" {
      + bucket = "datacenter-logs-2121"
      + id     = (known after apply)
    }

Plan: 6 to add, 0 to change, 0 to destroy.
```

**Success Indicators:**
- ✅ Data source resolved immediately (`ami-0abcd...` shown in plan)
- ✅ All 6 resources planned for creation
- ✅ IAM policy `Resource` references the correct bucket ARN
- ✅ EC2 instance shows the instance profile name

---

### Step 11: Apply Configuration

```bash
terraform apply -auto-approve
```

**Expected Output:**
```
Plan: 6 to add, 0 to change, 0 to destroy.

aws_s3_bucket.logs_bucket: Creating...
aws_s3_bucket.logs_bucket: Creation complete after 2s [id=datacenter-logs-2121]
aws_iam_role.datacenter_role: Creating...
aws_iam_role.datacenter_role: Creation complete after 2s [id=datacenter-role]
aws_iam_policy.datacenter_policy: Creating...
aws_iam_policy.datacenter_policy: Creation complete after 1s [id=arn:aws:iam::123456789012:policy/datacenter-access-policy]
aws_iam_role_policy_attachment.attach_policy: Creating...
aws_iam_role_policy_attachment.attach_policy: Creation complete after 1s [id=datacenter-role-arn:aws:iam::123456789012:policy/datacenter-access-policy]
aws_iam_instance_profile.datacenter_profile: Creating...
aws_iam_instance_profile.datacenter_profile: Creation complete after 2s [id=datacenter-instance-profile]
aws_instance.datacenter_ec2: Creating...
aws_instance.datacenter_ec2: Still creating... [10s elapsed]
aws_instance.datacenter_ec2: Creation complete after 32s [id=i-0datacenter111111]

Apply complete! Resources: 6 added, 0 changed, 0 destroyed.
```

**Success Indicators:**
- ✅ All 6 resources created in correct dependency order
- ✅ EC2 instance created last (depends on profile, which depends on role)
- ✅ `6 added, 0 changed, 0 destroyed`

---

### Step 12: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Expected Output:**
```
data.aws_ami.amazon_linux_2: Reading...
data.aws_ami.amazon_linux_2: Read complete after 1s [id=ami-0abcd1234efgh5678]
aws_s3_bucket.logs_bucket: Refreshing state... [id=datacenter-logs-2121]
aws_iam_role.datacenter_role: Refreshing state... [id=datacenter-role]
aws_iam_policy.datacenter_policy: Refreshing state... [id=arn:aws:iam::123456789012:policy/datacenter-access-policy]
aws_iam_role_policy_attachment.attach_policy: Refreshing state...
aws_iam_instance_profile.datacenter_profile: Refreshing state...
aws_instance.datacenter_ec2: Refreshing state... [id=i-0datacenter111111]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**⚠️ Required before task submission.**

---

## ✅ Verification Steps

### Step 1: Verify All Resources in State

```bash
terraform state list
```

**Expected Output:**
```
data.aws_ami.amazon_linux_2
aws_iam_instance_profile.datacenter_profile
aws_iam_policy.datacenter_policy
aws_iam_role.datacenter_role
aws_iam_role_policy_attachment.attach_policy
aws_instance.datacenter_ec2
aws_s3_bucket.logs_bucket
```

---

### Step 2: Verify IAM Role Trust Policy

```bash
terraform state show aws_iam_role.datacenter_role
```

**Expected Output (excerpt):**
```
assume_role_policy = jsonencode({
    Statement = [
        {
            Action    = "sts:AssumeRole"
            Effect    = "Allow"
            Principal = {
                Service = "ec2.amazonaws.com"
            }
        },
    ]
    Version = "2012-10-17"
})
name = "datacenter-role"
```

---

### Step 3: Verify IAM Policy Permissions

```bash
terraform state show aws_iam_policy.datacenter_policy
```

**Expected Output (excerpt):**
```
name   = "datacenter-access-policy"
policy = jsonencode({
    Statement = [
        {
            Action   = ["s3:PutObject"]
            Effect   = "Allow"
            Resource = "arn:aws:s3:::datacenter-logs-2121/*"
        },
    ]
    Version = "2012-10-17"
})
```

**Verification Points:**
- ✅ `Action = "s3:PutObject"` only (least privilege)
- ✅ `Resource` ends with `/*` (objects only, not the bucket itself)

---

### Step 4: Verify EC2 Has Instance Profile

```bash
terraform state show aws_instance.datacenter_ec2
```

**Expected Output (excerpt):**
```
ami                  = "ami-0abcd1234efgh5678"
iam_instance_profile = "datacenter-instance-profile"
instance_type        = "t2.micro"
tags = {
    "Name" = "datacenter-ec2"
}
```

---

### Step 5: AWS CLI Verification (Optional)

```bash
# Verify instance profile is attached to EC2
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].{ID:InstanceId,Profile:IamInstanceProfile.Arn}" \
  --output table

# Verify IAM role has the policy attached
aws iam list-attached-role-policies \
  --role-name datacenter-role \
  --query "AttachedPolicies[].{PolicyName:PolicyName,PolicyArn:PolicyArn}" \
  --output table
```

---

## 🔍 Code Analysis

### Full Resource Dependency Graph

```
data.aws_ami.amazon_linux_2         aws_s3_bucket.logs_bucket
        │                                   │
        │ .id (AMI ID)                      │ .arn (bucket ARN)
        ▼                                   ▼
aws_instance.datacenter_ec2      aws_iam_policy.datacenter_policy
        │ iam_instance_profile              │ .arn (policy ARN)
        ▲                                   ▼
aws_iam_instance_profile          aws_iam_role_policy_attachment
        │ role                              ▲
        └──────────────────────────────────┤
                                           │ .name
                              aws_iam_role.datacenter_role
```

### IAM Trust Policy vs Access Policy

| Policy Type | Resource | Purpose | Who It Controls |
|-------------|----------|---------|----------------|
| **Trust Policy** (`assume_role_policy`) | `aws_iam_role` | Who can assume this role | EC2 service (`ec2.amazonaws.com`) |
| **Access Policy** (`aws_iam_policy`) | `aws_iam_policy` | What the role can do | Actions on AWS resources |

```hcl
# Trust Policy — WHO can use this role
Principal = { Service = "ec2.amazonaws.com" }
Action    = "sts:AssumeRole"

# Access Policy — WHAT the role can do
Action   = ["s3:PutObject"]
Resource = "arn:aws:s3:::datacenter-logs-2121/*"
```

### Why `Resource` Ends with `/*`

```
arn:aws:s3:::datacenter-logs-2121     ← The BUCKET itself
arn:aws:s3:::datacenter-logs-2121/*   ← OBJECTS inside the bucket ✅
```

`s3:PutObject` operates on **objects**, not the bucket. The `/*` wildcard allows writing any object into the bucket. Without `/*`, the policy would target the bucket ARN itself (used for `s3:CreateBucket`, `s3:ListBucket`, etc.) — and `PutObject` would never work.

### IAM Role vs Instance Profile

```
IAM Role                            Instance Profile
────────                            ────────────────
aws_iam_role.datacenter_role   →    aws_iam_instance_profile.datacenter_profile
  (the permissions container)         (EC2-specific wrapper for the role)

EC2 cannot attach IAM Roles directly.
EC2 attaches Instance Profiles, which contain exactly one IAM Role.
```

### `data.aws_ami` — Dynamic AMI Resolution

```hcl
data "aws_ami" "amazon_linux_2" {
  most_recent = true          # Pick newest when multiple match
  owners      = ["amazon"]   # Only official Amazon AMIs

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
    #          ↑             ↑       ↑   ↑
    #          Amazon        any     64  EBS General Purpose SSD
    #          Linux 2       version bit root volume
  }
}
# Result: data.aws_ami.amazon_linux_2.id → e.g., "ami-0abcd1234efgh5678"
```

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate all files
terraform validate

# 2. Preview all 6 resources
terraform plan

# 3. Apply
terraform apply -auto-approve

# 4. Verify state has all 7 items (6 resources + 1 data source)
terraform state list

# 5. Verify IAM role trust policy
terraform state show aws_iam_role.datacenter_role

# 6. Verify IAM policy grants s3:PutObject only
terraform state show aws_iam_policy.datacenter_policy

# 7. Verify EC2 has instance profile attached
terraform state show aws_instance.datacenter_ec2

# 8. Final plan — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| EC2 name | `datacenter-ec2` | `terraform state show aws_instance.datacenter_ec2` |
| EC2 AMI | Latest Amazon Linux 2 | `terraform state show aws_instance.datacenter_ec2` |
| Instance profile on EC2 | `datacenter-instance-profile` | `terraform state show aws_instance.datacenter_ec2` |
| IAM role name | `datacenter-role` | `terraform state show aws_iam_role.datacenter_role` |
| Trust principal | `ec2.amazonaws.com` | `terraform state show aws_iam_role.datacenter_role` |
| Policy name | `datacenter-access-policy` | `terraform state show aws_iam_policy.datacenter_policy` |
| Policy action | `s3:PutObject` only | `terraform state show aws_iam_policy.datacenter_policy` |
| Policy resource | `arn:aws:s3:::datacenter-logs-2121/*` | `terraform state show aws_iam_policy.datacenter_policy` |
| S3 bucket name | `datacenter-logs-2121` | `terraform state show aws_s3_bucket.logs_bucket` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: AMI Not Found

**Error:**
```
Error: no matching AMI found
```

**Cause:** The AMI filter in `data.tf` doesn't match any AMIs in the target region, or the region has different naming conventions.

**Solution:**
```bash
# Search for available Amazon Linux 2 AMIs in your region
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" \
  --query "Images[0].Name" \
  --output text

# Update the filter in data.tf if the naming pattern differs
```

---

### Issue 2: IAM Role Already Exists

**Error:**
```
Error: creating IAM Role: EntityAlreadyExists: Role with name datacenter-role already exists.
```

**Solution:**
```bash
# Import existing role into Terraform state
terraform import aws_iam_role.datacenter_role datacenter-role

# Then run plan
terraform plan   # Should show "No changes" or minor drift
```

---

### Issue 3: Policy Resource Requires `/*` Suffix

**Error:**
```
Error: creating IAM Policy: MalformedPolicyDocument: Action does not apply to any resource(s) in statement
```

**Cause:** `s3:PutObject` requires a resource ARN ending with `/*`. Without it, the resource references the bucket itself, not its objects.

**Solution:**
```hcl
# ❌ Wrong — targets the bucket, not objects
Resource = aws_s3_bucket.logs_bucket.arn

# ✅ Correct — targets objects within the bucket
Resource = "${aws_s3_bucket.logs_bucket.arn}/*"
```

---

### Issue 4: EC2 Cannot Assume Role

**Symptom:** EC2 instance cannot write to S3 even though the policy is attached.

**Common Causes and Checks:**
```bash
# 1. Verify trust policy allows ec2.amazonaws.com
aws iam get-role --role-name datacenter-role \
  --query "Role.AssumeRolePolicyDocument"

# 2. Verify the policy is attached to the role
aws iam list-attached-role-policies --role-name datacenter-role

# 3. Verify instance profile is attached to the EC2 instance
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[0].Instances[0].IamInstanceProfile.Arn"
```

---

### Issue 5: Plan Shows AMI Change Every Apply

**Symptom:**
```
~ aws_instance.datacenter_ec2
  ~ ami = "ami-old" → "ami-new"
```

**Cause:** Amazon published a new Amazon Linux 2 AMI after the initial apply. `most_recent = true` now returns a newer AMI ID.

**Solution:**
```bash
# Option A: Ignore AMI drift (instance is running fine)
# Add to main.tf aws_instance block:
lifecycle {
  ignore_changes = [ami]
}

# Option B: Accept the change — plan will show recreation
# This will destroy and recreate the EC2 instance with the newer AMI
terraform apply -auto-approve
```

---

## 📚 Best Practices

### 1. Always Use IAM Roles Over Access Keys for EC2
Embedding AWS access keys in EC2 instances is a major security risk. IAM roles with instance profiles provide **short-lived, automatically rotated credentials** via IMDS — no key management required.

### 2. Apply Least Privilege to IAM Policies
Grant only the minimum permissions needed. For log uploading:
```hcl
# ✅ Least privilege — upload only
Action   = ["s3:PutObject"]

# ❌ Overly permissive — unnecessary access
Action   = ["s3:*"]
```

### 3. Scope Policy Resource to Specific Prefix (Advanced)
For production, further restrict to a specific prefix:
```hcl
Resource = "${aws_s3_bucket.logs_bucket.arn}/logs/ec2/*"
```
This prevents the EC2 instance from writing to other paths in the bucket.

### 4. Use Dynamic AMI Resolution in `data.tf`
Never hardcode AMI IDs. Data sources ensure each region and each deployment uses the current, patched Amazon Linux 2 image.

### 5. Use `jsonencode()` for Inline IAM Policies
`jsonencode()` produces valid JSON from native HCL maps — no manual escaping, no heredoc strings. It's readable, validates at plan time, and integrates naturally with other resource references.

### 6. Consider `lifecycle { ignore_changes = [ami] }` for Stable Instance Fleets
In production, AMI updates should be controlled via re-imaging processes, not automatic Terraform drift detection. Add `ignore_changes = [ami]` to prevent unplanned instance replacements when Amazon publishes new AMIs.

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Name | Role in Architecture |
|----------|------|---------------------|
| S3 Bucket | `datacenter-logs-2121` | Log destination |
| IAM Role | `datacenter-role` | EC2 identity (trust: `ec2.amazonaws.com`) |
| IAM Policy | `datacenter-access-policy` | `s3:PutObject` on bucket objects |
| Policy Attachment | — | Role ← Policy link |
| Instance Profile | `datacenter-instance-profile` | EC2-compatible role wrapper |
| EC2 Instance | `datacenter-ec2` | Log-producing instance with S3 write access |
| AMI Data Source | `amazon_linux_2` | Dynamic latest Amazon Linux 2 AMI |

### Files Created

| File | Purpose |
|------|---------|
| `main.tf` | All 6 AWS resources (S3, IAM role, policy, attachment, profile, EC2) |
| `data.tf` | `aws_ami` data source for latest Amazon Linux 2 |
| `variables.tf` | `KKE_BUCKET_NAME`, `KKE_POLICY_NAME`, `KKE_ROLE_NAME` declarations |
| `terraform.tfvars` | Variable values (`datacenter-logs-2121`, `datacenter-access-policy`, `datacenter-role`) |

### Task Completion Checklist

- [x] ✅ S3 bucket `datacenter-logs-2121` created
- [x] ✅ IAM role `datacenter-role` created with EC2 trust policy
- [x] ✅ IAM policy `datacenter-access-policy` grants `s3:PutObject` on bucket only
- [x] ✅ Policy attached to role via `aws_iam_role_policy_attachment`
- [x] ✅ Instance profile `datacenter-instance-profile` wraps the role for EC2 use
- [x] ✅ EC2 instance `datacenter-ec2` created with dynamic Amazon Linux 2 AMI and instance profile
- [x] ✅ All resources defined in **single `main.tf`** file
- [x] ✅ `data.tf` dynamically resolves latest Amazon Linux 2 AMI
- [x] ✅ `variables.tf` declares `KKE_BUCKET_NAME`, `KKE_POLICY_NAME`, `KKE_ROLE_NAME`
- [x] ✅ `terraform.tfvars` supplies all variable values
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform apply -auto-approve` — **6 resources added**
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task implements a **secure, credential-free EC2-to-S3 log pipeline** using the full IAM chain: IAM Role → Policy Attachment → Instance Profile → EC2. The key insight is that **EC2 instances cannot directly use IAM roles** — roles must be wrapped in an Instance Profile first. The `jsonencode()` function keeps inline IAM policies readable and type-safe within HCL. Dynamic AMI resolution via `data "aws_ami"` ensures the instance always launches with the latest patched Amazon Linux 2 image — eliminating hardcoded AMI ID maintenance across regions and time.
