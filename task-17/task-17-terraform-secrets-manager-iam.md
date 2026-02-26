# 🌟 Task 17 - Provision AWS Secrets Manager Secret and Inline IAM Access Policy Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is implementing secure secret management for cloud-native applications. The objective is to store sensitive database credentials in **AWS Secrets Manager** and configure an **IAM role** that allows EC2 instances to retrieve these secrets securely. This configuration uses an **inline IAM policy** to grant granular `GetSecretValue` permissions, ensuring that only specialized application roles can access sensitive production data.

**Requirements:**
- Create a **secret** in AWS Secrets Manager named **`devops-app-secret`**.
- Store the following **secret string** (JSON): `{"db_user":"admin","db_pass":"supersecret"}`.
- Create an **IAM role** named **`devops-app-role`** with **EC2** as the trusted entity (assume role policy).
- Attach an **inline IAM policy** named **`devops-app-policy`** to the role.
  - Permission: **`secretsmanager:GetSecretValue`**.
  - Resource: Scoped strictly to the **ARN of the `devops-app-secret`**.
- All resources must be defined in a single **`main.tf`** file.
- Use **`variables.tf`** to declare:
  - `KKE_SECRET_NAME`: Name of the secret.
  - `KKE_SECRET_VALUE`: The secret string value.
  - `KKE_ROLE_NAME`: Name of the IAM role.
  - `KKE_POLICY_NAME`: Name of the inline IAM policy.
- Use **`terraform.tfvars`** to provide the actual values for these variables.
- Use **`outputs.tf`** to export:
  - `KKE_secret_name`: The secret name.
  - `KKE_role_name`: The IAM role name.
  - `KKE_policy_name`: The inline IAM policy name.
- The Terraform working directory is **`/home/bob/terraform`**.
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Build a secure secret-access pipeline using Terraform — provisioning a centralized secret and the required IAM identity and inline policy to allow secure retrieval by authorized services.

💡 **Note:** Unlike managed policies, **inline policies** (`aws_iam_role_policy`) are embedded directly into a single IAM role. They are used for policies that have a strict 1-to-1 relationship with a specific role and should be deleted if the role itself is deleted.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- Secrets Manager Secret (`devops-app-secret`)
- Secrets Manager Secret Version (stores the credential JSON)
- IAM Role (`devops-app-role`) — trusted by EC2
- IAM Inline Policy (`devops-app-policy`) — scoped `GetSecretValue` permission

**Working Directory:** `/home/bob/terraform`

**Secret Retrieval Architecture:**
```
EC2 Instance / Application
        │ 
        ▼ assume_role (Trust: ec2.amazonaws.com)
IAM Role (devops-app-role)
        │
        │ inline policy
        ▼
IAM Inline Policy (devops-app-policy)
        │
        ▼ grants
action: secretsmanager:GetSecretValue ──► resource: [devops-app-secret ARN]
```

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_secretsmanager_secret`:** Defines the secret metadata (name, tags).
- **`aws_secretsmanager_secret_version`:** Manages the actual sensitive data (secret string) associated with the secret.
- **`aws_iam_role`:** Provisions the IAM role with the correct trust relationship.
- **`aws_iam_role_policy`:** Provisions the **inline** policy. Note the difference from `aws_iam_policy` (which creates a standalone managed policy).
- **Variable-Driven Design:** All sensitive values and names are stored in `variables.tf` and `terraform.tfvars`, following infrastructure-as-code best practices.

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # All resources (Secret, Role, Inline Policy)
├── variables.tf       # Declarations for secret and role variables
├── terraform.tfvars   # Actual assignments including JSON secret string
└── outputs.tf         # Outputs for verification
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

---

### Step 2: Create Variables Definition File

Create `variables.tf` to declare the four required variables:

```bash
cat > variables.tf << 'EOF'
variable "KKE_SECRET_NAME" {
  type = string
}

variable "KKE_SECRET_VALUE" {
  type = string
}

variable "KKE_ROLE_NAME" {
  type = string
}

variable "KKE_POLICY_NAME" {
  type = string
}
EOF
```

---

### Step 3: Create Terraform Variables File

Create `terraform.tfvars`. Be careful with JSON escaping in the secret string:

```bash
cat > terraform.tfvars << 'EOF'
KKE_SECRET_NAME  = "devops-app-secret"
KKE_SECRET_VALUE = "{\"db_user\":\"admin\",\"db_pass\":\"supersecret\"}"
KKE_ROLE_NAME    = "devops-app-role"
KKE_POLICY_NAME  = "devops-app-policy"
EOF
```

---

### Step 4: Create Main Configuration File

Create `main.tf` with the Secret, Role, and Inline Policy:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# Create Secret
# ---------------------------
resource "aws_secretsmanager_secret" "app_secret" {
  name = var.KKE_SECRET_NAME
}

resource "aws_secretsmanager_secret_version" "app_secret_value" {
  secret_id     = aws_secretsmanager_secret.app_secret.id
  secret_string = var.KKE_SECRET_VALUE
}

# ---------------------------
# IAM Role (Trusted by EC2)
# ---------------------------
resource "aws_iam_role" "app_role" {
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
# Inline IAM Policy (Read Secret Only)
# ---------------------------
resource "aws_iam_role_policy" "app_inline_policy" {
  name = var.KKE_POLICY_NAME
  role = aws_iam_role.app_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = "secretsmanager:GetSecretValue"
        Resource = aws_secretsmanager_secret.app_secret.arn
      }
    ]
  })
}
EOF
```

---

### Step 5: Create Outputs Configuration File

Create `outputs.tf` for verification:

```bash
cat > outputs.tf << 'EOF'
output "KKE_secret_name" {
  value = aws_secretsmanager_secret.app_secret.name
}

output "KKE_role_name" {
  value = aws_iam_role.app_role.name
}

output "KKE_policy_name" {
  value = aws_iam_role_policy.app_inline_policy.name
}
EOF
```

---

### Step 6: Initialize, Validate, and Apply

```bash
# Initialize Terraform
terraform init

# Validate syntax
terraform validate

# Format configuration
terraform fmt

# Apply the execution plan
terraform apply -auto-approve
```

**Expected Success Message:**
```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

KKE_policy_name = "devops-app-policy"
KKE_role_name = "devops-app-role"
KKE_secret_name = "devops-app-secret"
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
aws_iam_role.app_role
aws_iam_role_policy.app_inline_policy
aws_secretsmanager_secret.app_secret
aws_secretsmanager_secret_version.app_secret_value
```

---

### Step 2: Verify Secret Value (Safe Check)

```bash
# Verify the secret exists and get its metadata
aws secretsmanager describe-secret --secret-id devops-app-secret
```

---

### Step 3: Inspect Inline Policy ARN Scoping

```bash
terraform state show aws_iam_role_policy.app_inline_policy
```

**Verify:** Check that the `Resource` ARN in the policy matches the ARN of the `app_secret`.

---

## 🔍 Code Analysis

### Inline vs. Managed Policies
In this task, we implemented an **inline policy** using `aws_iam_role_policy`.
- **Inline Policy:** Defined *inside* the role. Stays strictly tied to this identity. Good for "singular" permissions that won't be reused elsewhere.
- **Managed Policy:** Standalone resource (`aws_iam_policy`). Can be shared across multiple roles/users.

### Security Best Practice: Least Privilege
By granting strictly `secretsmanager:GetSecretValue` on the specific secret ARN, we ensure that if the instance is compromised, it cannot read other secrets or perform administrative actions (like deleting the secret).

---

## 🎯 Task Completion Summary

### Resources Created

| Resource | Value |
|----------|-------|
| Secret | `devops-app-secret` |
| Secret Version | `{"db_user":"admin","db_pass":"supersecret"}` |
| IAM Role | `devops-app-role` |
| Inline Policy | `devops-app-policy` (attached to Role) |

### Task Completion Checklist
- [x] ✅ AWS Secrets Manager Secret `devops-app-secret` provisioned.
- [x] ✅ JSON secret string stored successfully.
- [x] ✅ IAM role `devops-app-role` created with EC2 trust Relationship.
- [x] ✅ **Inline** IAM policy `devops-app-policy` created and attached.
- [x] ✅ Permissions strictly scoped to `GetSecretValue` on the specific ARN.
- [x] ✅ Use of `variables.tf` and `terraform.tfvars` for all inputs.
- [x] ✅ All logic contained within `main.tf`.
- [x] ✅ Final `terraform plan` returns **"No changes"**.
