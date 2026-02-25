# 🌟 Task 13 - Provision IAM Role and Policy Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is implementing IAM-based access control for internal AWS resources. The objective is to create an IAM Role and an IAM Policy using Terraform and then attach the policy to the role. This ensures that any entity assuming the role will have the specific permissions defined in the policy — in this case, the ability to list EC2 instances.

**Requirements:**
- Create an **IAM Role** named **`devops-role`**
- Create an **IAM Policy** named **`devops-policy`** that allows **`ec2:DescribeInstances`**
- **Attach the policy to the role** using a policy attachment resource
- All resources must be defined in a single **`main.tf`** file
- Use **`variables.tf`** to declare:
  - `KKE_ROLE_NAME` — name of the role
  - `KKE_POLICY_NAME` — name of the policy
- Use **`terraform.tfvars`** to provide the actual names for the role and policy
- Use **`outputs.tf`** to export:
  - `kke_iam_role_name` — name of the created role
  - `kke_iam_policy_name` — name of the created policy
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Build a foundational IAM access control structure — an IAM Role, a customer-managed IAM Policy, and an attachment that links them — using variable-driven configuration for flexibility and reuse.

💡 **Note:** IAM Roles require an `assume_role_policy` (Trust Policy) to define which principals can assume the role. For this task, we will configure the role to be assumable by the EC2 service (`ec2.amazonaws.com`).

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (IAM is a global service)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- IAM Role (`devops-role`) — the identity container
- IAM Policy (`devops-policy`) — the permission set (`ec2:DescribeInstances`)
- IAM Policy Attachment — the bridge between the role and policy

**Working Directory:** `/home/bob/terraform`

**IAM Permission Logic:**
```
Principal (e.g., EC2 Instance)
        │ 
        ▼ assume_role
IAM Role (devops-role)
        │
        │ attached policy
        ▼
IAM Policy (devops-policy)
        │ 
        ▼ allows
ec2:DescribeInstances on Resource: "*"
```

**Resource Summary:**
| Resource | Terraform Type | Name Source |
|----------|---------------|-------------|
| IAM Role | `aws_iam_role` | `var.KKE_ROLE_NAME` |
| IAM Policy | `aws_iam_policy` | `var.KKE_POLICY_NAME` |
| Policy Attachment | `aws_iam_role_policy_attachment` | — |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_iam_role`:** Creates the IAM Role with a trust policy allowing the EC2 service to assume it.
- **`aws_iam_policy`:** Creates a customer-managed policy with the `ec2:DescribeInstances` action.
- **`aws_iam_role_policy_attachment`:** Links the policy ARN to the role name.
- **`variables.tf` & `terraform.tfvars`:** Decouples resource names from the logic, allowing for easy environment-specific overrides.
- **`outputs.tf`:** Provides immediate visibility into the created resources after application.

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # Logic for Role, Policy, and Attachment
├── variables.tf       # Declaration of role and policy name variables
├── terraform.tfvars   # Assignment of actual values ("devops-role", "devops-policy")
└── outputs.tf         # Outputs for role and policy names
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

---

### Step 2: Create Variables Definition File

Create `variables.tf` to declare the role and policy name variables:

```bash
cat > variables.tf << 'EOF'
variable "KKE_ROLE_NAME" {
  type = string
}

variable "KKE_POLICY_NAME" {
  type = string
}
EOF
```

**Configuration Breakdown:**
- Two string variables declared without defaults, ensuring values must be provided at runtime (via tfvars or CLI).

**File Content (`variables.tf`):**
```hcl
variable "KKE_ROLE_NAME" {
  type = string
}

variable "KKE_POLICY_NAME" {
  type = string
}
```

---

### Step 3: Create Terraform Variables File

Create `terraform.tfvars` to assign values to the variables:

```bash
cat > terraform.tfvars << 'EOF'
KKE_ROLE_NAME   = "devops-role"
KKE_POLICY_NAME = "devops-policy"
EOF
```

**File Content (`terraform.tfvars`):**
```hcl
KKE_ROLE_NAME   = "devops-role"
KKE_POLICY_NAME = "devops-policy"
```

---

### Step 4: Create Main Configuration File

Create `main.tf` with the Role, Policy, and Attachment:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# IAM Role
# ---------------------------
resource "aws_iam_role" "devops_role" {
  name = var.KKE_ROLE_NAME

  # Trust Policy permitting EC2 service to assume this role
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
resource "aws_iam_policy" "devops_policy" {
  name = var.KKE_POLICY_NAME

  # Permission policy allowing EC2 list actions
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = "ec2:DescribeInstances"
        Resource = "*"
      }
    ]
  })
}

# ---------------------------
# Attach Policy to Role
# ---------------------------
resource "aws_iam_role_policy_attachment" "attach_policy" {
  role       = aws_iam_role.devops_role.name
  policy_arn = aws_iam_policy.devops_policy.arn
}
EOF
```

**Configuration Breakdown:**

**1. IAM Role (`aws_iam_role.devops_role`):**
- `name = var.KKE_ROLE_NAME` — Named dynamically from variables.
- `assume_role_policy` — Defines WHO can assume this role. Here, we use `jsonencode()` to create a clean JSON block allowing the **EC2 service** to assume the role.

**2. IAM Policy (`aws_iam_policy.devops_policy`):**
- `name = var.KKE_POLICY_NAME` — Named dynamically from variables.
- `policy` — Defines WHAT the role can do. Allows the `ec2:DescribeInstances` action on all resources (`"*"`).

**3. Policy Attachment (`aws_iam_role_policy_attachment.attach_policy`):**
- `role` — References the first resource by name.
- `policy_arn` — References the second resource's ARN.
- This creates the actual permission link.

**File Content (`main.tf`):**
```hcl
resource "aws_iam_role" "devops_role" {
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

resource "aws_iam_policy" "devops_policy" {
  name = var.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = "ec2:DescribeInstances"
        Resource = "*"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "attach_policy" {
  role       = aws_iam_role.devops_role.name
  policy_arn = aws_iam_policy.devops_policy.arn
}
```

---

### Step 5: Create Outputs Configuration File

Create `outputs.tf` to export the created names:

```bash
cat > outputs.tf << 'EOF'
output "kke_iam_role_name" {
  value = aws_iam_role.devops_role.name
}

output "kke_iam_policy_name" {
  value = aws_iam_policy.devops_policy.name
}
EOF
```

**File Content (`outputs.tf`):**
```hcl
output "kke_iam_role_name" {
  value = aws_iam_role.devops_role.name
}

output "kke_iam_policy_name" {
  value = aws_iam_policy.devops_policy.name
}
```

---

### Step 6: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Expected Output:**
```
total 20
drwxr-xr-x 2 bob bob 4096 Feb 25 15:30 .
drwxr-xr-x 5 bob bob 4096 Feb 25 15:25 ..
-rw-r--r-- 1 bob bob  945 Feb 25 15:30 main.tf
-rw-r--r-- 1 bob bob  155 Feb 25 15:30 outputs.tf
-rw-r--r-- 1 bob bob   65 Feb 25 15:30 terraform.tfvars
-rw-r--r-- 1 bob bob  112 Feb 25 15:30 variables.tf
```

---

### Step 7: Initialize, Validate, and Apply

```bash
# Initialize
terraform init

# Validate
terraform validate

# Format
terraform fmt

# Apply
terraform apply -auto-approve
```

**Expected Success Message:**
```
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

kke_iam_policy_name = "devops-policy"
kke_iam_role_name = "devops-role"
```

---

### Step 8: Final Verification (Critical)

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

### Step 1: Verify Resources in State

```bash
terraform state list
```

**Expected Output:**
```
aws_iam_policy.devops_policy
aws_iam_role.devops_role
aws_iam_role_policy_attachment.attach_policy
```

---

### Step 2: Inspect IAM Role Trust Policy

```bash
terraform state show aws_iam_role.devops_role
```

**Verify:** Principal is `"Service": "ec2.amazonaws.com"`.

---

### Step 3: Inspect IAM Policy Contents

```bash
terraform state show aws_iam_policy.devops_policy
```

**Verify:** Action includes `"ec2:DescribeInstances"`.

---

## 🔍 Code Analysis

### IAM Policy Design: Inline vs Customer-Managed

| Type | Resource | Usage in this Task |
|------|----------|-------------------|
| **Inline Policy** | `aws_iam_role_policy` | ❌ Not used (Harder to manage/share) |
| **Managed Policy** | `aws_iam_policy` | ✅ Used (Reusable, standalone) |
| **Trust Policy** | `aws_iam_role` | ✅ Used (Mandatory for roles) |

### `jsonencode()` Benefit
Using `jsonencode()` is preferred over heredoc strings because:
- ✅ Avoids manual JSON escaping issues.
- ✅ Allows standard HCL syntax for comments and variable interpolation.
- ✅ Produces minified/standardized JSON.

---

## 🎯 Task Completion Summary

### Resources Created

| Resource | Value |
|----------|-------|
| IAM Role | `devops-role` |
| IAM Policy | `devops-policy` |
| Attachment | Link (Role 🔗 Policy) |

### Task Completion Checklist
- [x] ✅ Created IAM Role `devops-role` with EC2 trust policy.
- [x] ✅ Created IAM Policy `devops-policy` for listing EC2 instances.
- [x] ✅ Attached policy to the role successfully.
- [x] ✅ Logic centralized in a single `main.tf`.
- [x] ✅ Variable-driven naming via `variables.tf` and `terraform.tfvars`.
- [x] ✅ Outputs verified for role and policy names.
- [x] ✅ Final `terraform plan` returns **"No changes"**.
