# 🌟 Task 16 - Provision SNS Topic and IAM Role for Message Publishing Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is setting up a secure messaging infrastructure to enable inter-service communication. The objective is to create an **SNS topic** and configure an **IAM role** that allows EC2 instances to publish messages to that specific topic. By using a scoped IAM policy and a trust relationship, the team ensures that only authorized entities can interact with the messaging system.

**Requirements:**
- Create an **SNS topic** named **`nautilus-sns-topic`**
- Create an **IAM role** named **`nautilus-sns-role`** with **EC2** as the trusted entity (assume role policy)
- Create an **IAM policy** named **`nautilus-sns-policy`** that grants **`sns:Publish`** permission on the topic ARN
- **Attach the policy to the role** using a policy attachment resource
- All resources must be defined in a single **`main.tf`** file (no separate `.tf` file for resources)
- Use **`locals.tf`** to define the following names:
  - `KKE_SNS_TOPIC_NAME`: `nautilus-sns-topic`
  - `KKE_ROLE_NAME`: `nautilus-sns-role`
  - `KKE_POLICY_NAME`: `nautilus-sns-policy`
- Use **`outputs.tf`** to export:
  - `kke_sns_topic_name`: name of the created SNS topic
  - `kke_role_name`: name of the created IAM role
  - `kke_policy_name`: name of the created IAM policy
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Build a secure AWS messaging channel using Terraform — provisioning an SNS topic and the IAM identity required to publish messages to it securely.

💡 **Note:** Using `locals` for resource naming is an excellent way to maintain a "single source of truth" for constants within your Terraform module, especially when those names are reused across different resources or policies.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- SNS Topic (`nautilus-sns-topic`) — the message dispatcher
- IAM Role (`nautilus-sns-role`) — the identity for EC2 publishing
- IAM Policy (`nautilus-sns-policy`) — `sns:Publish` permission
- IAM Policy Attachment — link between role and policy

**Working Directory:** `/home/bob/terraform`

**Messaging Security Architecture:**
```
EC2 Instance / Service
        │ 
        ▼ assume_role (Trust: ec2.amazonaws.com)
IAM Role (nautilus-sns-role)
        │
        │ attached policy
        ▼
IAM Policy (nautilus-sns-policy)
        │
        ▼ grants
action: sns:Publish ──► resource: [nautilus-sns-topic ARN]
```

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_sns_topic`:** The core messaging resource.
- **`aws_iam_role`:** Includes an `assume_role_policy` written in JSON that permits the Amazon EC2 service to assume this role.
- **`aws_iam_policy`:** Defines the specific `sns:Publish` permission. The `Resource` attribute dynamically references the SNS topic's ARN to ensure strictly scoped access.
- **`aws_iam_role_policy_attachment`:** Links the role and policy together.
- **`locals.tf`:** Centralizes name management, making the code easier to read and maintain.

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # Logic for SNS, IAM Role, Policy, and Attachment
├── locals.tf          # Local constants for resource naming
└── outputs.tf         # Outputs for verification
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

---

### Step 2: Create Locals Configuration File

Create `locals.tf` to define the naming constants:

```bash
cat > locals.tf << 'EOF'
locals {
  KKE_SNS_TOPIC_NAME = "nautilus-sns-topic"
  KKE_ROLE_NAME      = "nautilus-sns-role"
  KKE_POLICY_NAME    = "nautilus-sns-policy"
}
EOF
```

---

### Step 3: Create Main Configuration File

Create `main.tf` with the SNS topic, IAM role, and policy resources:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# SNS Topic
# ---------------------------
resource "aws_sns_topic" "nautilus_topic" {
  name = local.KKE_SNS_TOPIC_NAME
}

# ---------------------------
# IAM Role (Trusted by EC2)
# ---------------------------
resource "aws_iam_role" "nautilus_role" {
  name = local.KKE_ROLE_NAME

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
# IAM Policy (Allow Publish)
# ---------------------------
resource "aws_iam_policy" "nautilus_policy" {
  name = local.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = "sns:Publish"
        Resource = aws_sns_topic.nautilus_topic.arn
      }
    ]
  })
}

# ---------------------------
# Attach Policy to Role
# ---------------------------
resource "aws_iam_role_policy_attachment" "attach_policy" {
  role       = aws_iam_role.nautilus_role.name
  policy_arn = aws_iam_policy.nautilus_policy.arn
}
EOF
```

**Configuration Breakdown:**
- `local.NAME`: References variables defined in `locals.tf`.
- `jsonencode`: Standard method to convert HCL maps into valid JSON blocks for IAM policies.
- `Resource = aws_sns_topic.nautilus_topic.arn`: Ensures the role has permissions **only** for this specific topic.

---

### Step 4: Create Outputs Configuration File

Create `outputs.tf` to report the created resource names:

```bash
cat > outputs.tf << 'EOF'
output "kke_sns_topic_name" {
  value = aws_sns_topic.nautilus_topic.name
}

output "kke_role_name" {
  value = aws_iam_role.nautilus_role.name
}

output "kke_policy_name" {
  value = aws_iam_policy.nautilus_policy.name
}
EOF
```

---

### Step 5: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Expected Output:**
```
total 20
drwxr-xr-x 2 bob bob 4096 Feb 26 20:25 .
drwxr-xr-x 5 bob bob 4096 Feb 26 20:20 ..
-rw-r--r-- 1 bob bob  132 Feb 26 20:25 locals.tf
-rw-r--r-- 1 bob bob 1184 Feb 26 20:25 main.tf
-rw-r--r-- 1 bob bob  225 Feb 26 20:25 outputs.tf
```

---

### Step 6: Initialize, Validate, and Apply

```bash
# Initialize Terraform
terraform init

# Validate configuration syntax
terraform validate

# Format the code
terraform fmt

# Apply the execution plan
terraform apply -auto-approve
```

**Expected Success Message:**
```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

kke_policy_name = "nautilus-sns-policy"
kke_role_name = "nautilus-sns-role"
kke_sns_topic_name = "nautilus-sns-topic"
```

---

### Step 7: Final Verification (Critical)

```bash
terraform plan
```

**Expected Output:**
```
No changes. Your infrastructure matches the configuration.
```

✅ **Task complete.**

---

## ✅ Verification Steps

### Step 1: Verify State List

```bash
terraform state list
```

**Expected Output:**
```
aws_iam_policy.nautilus_policy
aws_iam_role.nautilus_role
aws_iam_role_policy_attachment.attach_policy
aws_sns_topic.nautilus_topic
```

---

### Step 2: Inspect IAM Role Trust Relationship

```bash
terraform state show aws_iam_role.nautilus_role
```

---

### Step 3: Inspect IAM Policy Scoping

```bash
terraform state show aws_iam_policy.nautilus_policy
```

**Verify:** The `Resource` ARN in the policy should match the ARN of the `nautilus_topic`.

---

## 🔍 Code Analysis

### Dynamic Resource Reference
In the `aws_iam_policy` resource, we use:
`Resource = aws_sns_topic.nautilus_topic.arn`

This is a **computed dependency**. Terraform knows it cannot create the policy until the SNS topic is provisioned (because the ARN is only known after creation). This ensures proper orchestration of resource provisioning.

### Role Trust Policy Explained
`Action = "sts:AssumeRole"` in the role block tells AWS IAM that the role identity can be "assumed" (activated). The `Principal = { Service = "ec2.amazonaws.com" }` restricts this activation to only the EC2 service. This prevents other services or users from impersonating the messaging role.

---

## 🎯 Task Completion Summary

### Resources Created

| Resource | Value |
|----------|-------|
| SNS Topic | `nautilus-sns-topic` |
| IAM Role | `nautilus-sns-role` |
| IAM Policy | `nautilus-sns-policy` |

### Task Completion Checklist
- [x] ✅ SNS topic `nautilus-sns-topic` provisioned.
- [x] ✅ IAM role `nautilus-sns-role` created with EC2 trust policy.
- [x] ✅ IAM policy `nautilus-sns-policy` created with `sns:Publish` permission.
- [x] ✅ Policy successfully attached to role.
- [x] ✅ `locals.tf` used for all resource naming constants.
- [x] ✅ `outputs.tf` provides real-time verification of resource names.
- [x] ✅ Logic centralized in `main.tf`.
- [x] ✅ `terraform validate` and `fmt` passed.
- [x] ✅ Final `terraform plan` returns **"No changes"**.
