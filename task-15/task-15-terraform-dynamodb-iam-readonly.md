# 🌟 Task 15 - Provision DynamoDB Table with Read-Only IAM Access Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is implementing secure access control for NoSQL databases. The goal is to create a **DynamoDB table** and enforce **fine-grained access control** using IAM. This involves creating a dedicated IAM role and a scoped read-only policy that limits access specifically to the newly created table, ensuring that only trusted services can perform read operations.

**Requirements:**
- Create a **DynamoDB Table** named **`nautilus-table`** with minimal configuration (Partition Key: `id`, type `S`).
- Create an **IAM Role** named **`nautilus-role`** (assignable by EC2 service).
- Create an **IAM Policy** named **`nautilus-readonly-policy`**:
  - Permissions: `GetItem`, `Scan`, `Query`.
  - Scoped strictly to the **ARN of the `nautilus-table`**.
- **Attach the policy to the role** using a policy attachment resource.
- All resources must be defined in a single **`main.tf`** file.
- Use **`variables.tf`** to declare:
  - `KKE_TABLE_NAME` — DynamoDB table name
  - `KKE_ROLE_NAME` — IAM role name
  - `KKE_POLICY_NAME` — IAM policy name
- Use **`terraform.tfvars`** to provide the actual values for these variables.
- Use **`outputs.tf`** to export:
  - `kke_dynamodb_table` — table name
  - `kke_iam_role_name` — role name
  - `kke_iam_policy_name` — policy name
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Implement a secure database access pattern — a DynamoDB table protected by a least-privilege IAM policy that restricts actions to specific read-only commands for that specific table resource.

💡 **Note:** Always use the resource ARN from the DynamoDB resource block (`aws_dynamodb_table.nautilus_table.arn`) in your IAM policy to ensure the policy is automatically updated if the table is recreated or changed.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- DynamoDB Table (`nautilus-table`) — NoSQL storage
- IAM Role (`nautilus-role`) — The identity for service access
- IAM Policy (`nautilus-readonly-policy`) — Read-only permissions on the table
- IAM Policy Attachment — Links permissions to the identity

**Working Directory:** `/home/bob/terraform`

**Access Control Logic:**
```
Trusted AWS Service (e.g. EC2) ──► Assume nautilus-role
                                          │
    ┌─────────────────────────────────────┘
    ▼
nautilus-readonly-policy attached
    │
    ├─ Allowed: dynamodb:GetItem, dynamodb:Scan, dynamodb:Query
    └─ Target: arn:aws:dynamodb:us-east-1:123456789012:table/nautilus-table
```

**Resource Summary:**
| Resource | Name | Key Features |
|----------|------|--------------|
| DynamoDB Table | `nautilus-table` | `id` (String) Partition Key, Pay-Per-Request |
| IAM Role | `nautilus-role` | Trust: `ec2.amazonaws.com` |
| IAM Policy | `nautilus-readonly-policy` | Read-only actions on table ARN |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_dynamodb_table`:** Configured with `billing_mode = "PAY_PER_REQUEST"` for cost efficiency and a simple string partition key `id`.
- **`aws_iam_role`:** Defined with an inline Trust Policy (`assume_role_policy`) allowing EC2 to use the role.
- **`aws_iam_policy`:** Uses `jsonencode()` to define a scoped policy. The `Resource` attribute points directly to the DynamoDB table's ARN.
- **`aws_iam_role_policy_attachment`:** The glue that connects the Role and Policy resources.
- **Variables & tfvars:** Enables a clean, professional separation between code logic and environment configuration.

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # Logic for DynamoDB, Role, Policy, and Attachment
├── variables.tf       # Variable declarations
├── terraform.tfvars   # Variable values (nautilus-table, nautilus-role, nautilus-readonly-policy)
└── outputs.tf         # Resource name outputs
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

---

### Step 2: Create Variables Definition File

Create `variables.tf` to declare the required inputs:

```bash
cat > variables.tf << 'EOF'
variable "KKE_TABLE_NAME" {
  description = "Name of the DynamoDB table"
  type        = string
}

variable "KKE_ROLE_NAME" {
  description = "Name of the IAM role"
  type        = string
}

variable "KKE_POLICY_NAME" {
  description = "Name of the IAM policy"
  type        = string
}
EOF
```

---

### Step 3: Create Terraform Variables File

Create `terraform.tfvars` to supply the specific resource names:

```bash
cat > terraform.tfvars << 'EOF'
KKE_TABLE_NAME  = "nautilus-table"
KKE_ROLE_NAME   = "nautilus-role"
KKE_POLICY_NAME = "nautilus-readonly-policy"
EOF
```

---

### Step 4: Create Main Configuration File

Create `main.tf` with the DynamoDB table, IAM role, and Policy:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# DynamoDB Table
# ---------------------------
resource "aws_dynamodb_table" "nautilus_table" {
  name         = var.KKE_TABLE_NAME
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

# ---------------------------
# IAM Role
# ---------------------------
resource "aws_iam_role" "nautilus_role" {
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
# IAM Policy (Read Only)
# ---------------------------
resource "aws_iam_policy" "readonly_policy" {
  name = var.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:Scan",
          "dynamodb:Query"
        ]
        Resource = aws_dynamodb_table.nautilus_table.arn
      }
    ]
  })
}

# ---------------------------
# Attach Policy to Role
# ---------------------------
resource "aws_iam_role_policy_attachment" "attach_policy" {
  role       = aws_iam_role.nautilus_role.name
  policy_arn = aws_iam_policy.readonly_policy.arn
}
EOF
```

**Configuration Breakdown:**
- `hash_key`: Defines the partition key for the NoSQL table.
- `billing_mode`: "PAY_PER_REQUEST" (On-Demand) avoids provisioning read/write capacity units.
- `Resource = aws_dynamodb_table.nautilus_table.arn`: Ensures the policy grants access **only** to this specific table ARN, fulfilling the request for fine-grained access.

---

### Step 5: Create Outputs Configuration File

Create `outputs.tf` to export names for verification:

```bash
cat > outputs.tf << 'EOF'
output "kke_dynamodb_table" {
  value = aws_dynamodb_table.nautilus_table.name
}

output "kke_iam_role_name" {
  value = aws_iam_role.nautilus_role.name
}

output "kke_iam_policy_name" {
  value = aws_iam_policy.readonly_policy.name
}
EOF
```

---

### Step 6: Initialize, Validate, and Apply

```bash
# Initialize
terraform init

# Validate syntax
terraform validate

# Format files
terraform fmt

# Apply changes
terraform apply -auto-approve
```

**Expected Success Message:**
```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

kke_dynamodb_table = "nautilus-table"
kke_iam_policy_name = "nautilus-readonly-policy"
kke_iam_role_name = "nautilus-role"
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

✅ **Task ready for submission.**

---

## ✅ Verification Steps

### Step 1: Verify Table and IAM Role in State

```bash
terraform state list
```

**Expected Output:**
```
aws_dynamodb_table.nautilus_table
aws_iam_policy.readonly_policy
aws_iam_role.nautilus_role
aws_iam_role_policy_attachment.attach_policy
```

---

### Step 2: Inspect DynamoDB Key Schema

```bash
terraform state show aws_dynamodb_table.nautilus_table
```

**Verify:** `hash_key = "id"` and `billing_mode = "PAY_PER_REQUEST"`.

---

### Step 3: Inspect Policy ARN Scoping

```bash
terraform state show aws_iam_policy.readonly_policy
```

**Verify:** The `Resource` matches the ARN of the `nautilus-table`.

---

## 🔍 Code Analysis

### Least-Privilege IAM Scoping

In this configuration, we avoided the easy but insecure `Resource = "*"` shortcut. By using the dynamic reference `Resource = aws_dynamodb_table.nautilus_table.arn`, we satisfy the security audit requirement:
1. **Action Scope:** Only read actions (`GetItem`, `Scan`, `Query`) are allowed.
2. **Resource Scope:** Only the specific `nautilus-table` is accessible.

### DynamoDB Billing Modes

| Mode | Usage | Benefit |
|------|-------|---------|
| **PROVISIONED** | Predictable traffic | Free tier eligibility, fixed cost |
| **PAY_PER_REQUEST** | Spiky/unknown traffic | ✅ Used here: Set and forget, pay only for what you use |

---

## 🎯 Task Completion Summary

### Resources Created
- **DynamoDB Table:** `nautilus-table`
- **IAM Role:** `nautilus-role`
- **IAM Managed Policy:** `nautilus-readonly-policy`
- **Attachment:** Links the policy to the role.

### Task Completion Checklist
- [x] ✅ Provisioned DynamoDB table `nautilus-table`.
- [x] ✅ Created IAM role `nautilus-role` (with EC2 trust).
- [x] ✅ Created and attached readonly IAM policy scoped to the table ARN.
- [x] ✅ Variables and tfvars used for resource naming.
- [x] ✅ All resources defined in `main.tf`.
- [x] ✅ Verified with `terraform plan` showing **"No changes"**.
