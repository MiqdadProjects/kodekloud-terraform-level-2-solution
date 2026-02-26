# 🌟 Task 20 - Provision CloudFormation Stack Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is exploring hybrid infrastructure-as-code patterns by automating **AWS CloudFormation** stack provisioning via **Terraform**. The objective is to deploy a CloudFormation stack that manages a DynamoDB table. This approach demonstrates how Terraform can orchestrate legacy or service-specific CloudFormation templates while maintaining overall state management in Terraform.

**Requirements:**
- Create a **CloudFormation stack** named **`nautilus-dynamodb-stack`**.
- The stack must deploy a **DynamoDB table** named **`nautilus-cf-dynamodb-table`** (defined within the CF template).
- Use the **`main.tf`** file (no separate files for resources) to provision the stack.
- Implement a **`lifecycle` block** in the `aws_cloudformation_stack` resource to **ignore changes to the `parameters` attribute**.
- Use **`variables.tf`** to declare:
  - `KKE_DYNAMODB_TABLE_NAME`: The name of the DynamoDB table.
- Use **`terraform.tfvars`** to provide the actual table name.
- Use **`locals.tf`** (already provided in the environment) which contains:
  - `cf_template_body`: A local variable storing the CloudFormation template JSON/YAML.
- Use **`outputs.tf`** to export:
  - `KKE_stack_name`: The name of the created CloudFormation stack.
- The Terraform working directory is **`/home/bob/terraform`**.
- Before submitting, ensure that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Orchestrate an AWS CloudFormation stack deployment using Terraform, leveraging locals for template management and the lifecycle meta-argument to prevent state drift from external parameter updates.

💡 **Note:** The `lifecycle { ignore_changes = [parameters] }` block is a critical part of this task. It ensures that if CloudFormation parameters are modified outside of Terraform (e.g., via the AWS Console), Terraform will not attempt to revert those specific changes on the next apply.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- CloudFormation Stack (`nautilus-dynamodb-stack`)
- DynamoDB Table (Created *inside* the stack: `nautilus-cf-dynamodb-table`)

**Working Directory:** `/home/bob/terraform`

**Infrastructure Orchestration Flow:**
```
[Terraform Configuration]
        │
        ▼ (Reads local.cf_template_body)
[aws_cloudformation_stack (nautilus-dynamodb-stack)]
        │
        ▼ (Stack Deployment)
[AWS CloudFormation Service]
        │
        ▼ (Resource Provisioning)
[DynamoDB Table: nautilus-cf-dynamodb-table]
```

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_cloudformation_stack`:** The Terraform resource that interfaces with the CloudFormation service. It passes the `template_body` from a local variable.
- **`lifecycle` block:** Used to manage how Terraform handles resource updates. By ignoring `parameters`, we allow for out-of-band parameter management if required.
- **`locals.tf`:** Serves as a clean container for the multi-line CloudFormation template body, keeping `main.tf` focused on the resource definition.
- **`variables.tf` & `tfvars`:** Standard input management for the table name variable used within the template or stack configuration.

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # CloudFormation stack resource + lifecycle
├── locals.tf          # CloudFormation template body definition
├── variables.tf       # Declaration of KKE_DYNAMODB_TABLE_NAME
├── terraform.tfvars   # Table name assignment
└── outputs.tf         # Stack name output
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

---

### Step 2: Create Variables Definition File

Create `variables.tf`:

```bash
cat > variables.tf << 'EOF'
variable "KKE_DYNAMODB_TABLE_NAME" {
  type = string
}
EOF
```

---

### Step 3: Create Terraform Variables File

Create `terraform.tfvars`:

```bash
cat > terraform.tfvars << 'EOF'
KKE_DYNAMODB_TABLE_NAME = "nautilus-cf-dynamodb-table"
EOF
```

---

### Step 4: Create Main Configuration File

Create `main.tf`. This identifies the resource and adds the required `lifecycle` rule:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# CloudFormation Stack
# ---------------------------
resource "aws_cloudformation_stack" "nautilus_stack" {
  name          = "nautilus-dynamodb-stack"
  template_body = local.cf_template_body

  # Lifecycle rule to ignore manual changes to parameters
  lifecycle {
    ignore_changes = [
      parameters
    ]
  }
}
EOF
```

**Configuration Breakdown:**
- `template_body`: Injected from the `local.cf_template_body` variable (defined in `locals.tf`).
- `ignore_changes = [parameters]`: Crucial requirement to prevent Terraform from fighting with manual or external CloudFormation parameter updates.

---

### Step 5: Create Outputs Configuration File

Create `outputs.tf`:

```bash
cat > outputs.tf << 'EOF'
output "KKE_stack_name" {
  value = aws_cloudformation_stack.nautilus_stack.name
}
EOF
```

---

### Step 6: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Expected Output include:**
`locals.tf` (already present), `main.tf`, `variables.tf`, `terraform.tfvars`, and `outputs.tf`.

---

### Step 7: Initialize, Validate, and Apply

```bash
# Initialize
terraform init

# Validate syntax
terraform validate

# Format
terraform fmt

# Apply
terraform apply -auto-approve
```

**Expected Success Message:**
```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

KKE_stack_name = "nautilus-dynamodb-stack"
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

### Step 1: Check Stack in Terraform State

```bash
terraform state show aws_cloudformation_stack.nautilus_stack
```

---

### Step 2: Verify Stack via AWS CLI

```bash
aws cloudformation describe-stacks --stack-name nautilus-dynamodb-stack
```

---

### Step 3: Confirm DynamoDB Table Creation

```bash
# List resources in the CloudFormation stack
aws cloudformation list-stack-resources --stack-name nautilus-dynamodb-stack
```

**Verify:** Look for the PhysicalResourceId matching `nautilus-cf-dynamodb-table`.

---

## 🔍 Code Analysis

### The `lifecycle` Meta-Argument
The `lifecycle` block is one of Terraform's most powerful configuration settings. 
- **`ignore_changes`**: Tells Terraform to skip certain attributes when calculating the diff for a plan. 
- **Why for Parameters?**: CloudFormation parameters might be updated by automated processes, CI/CD scripts, or manual overrides. Ignoring them in Terraform ensures that a subsequent `terraform apply` doesn't overwrite those specific runtime values.

### Why wrap CloudFormation in Terraform?
1. **Migration Path:** Easier to manage existing CF stacks during a migration to Terraform.
2. **Feature Parity:** Sometimes CloudFormation supports a new AWS feature slightly before the Terraform provider does.
3. **Internal Tooling:** If the organization has heavy investments in CloudFormation JSON/YAML templates, Terraform can serve as the top-level orchestrator.

---

## 🎯 Task Completion Summary

### Resources Created

| Resource | Value |
|----------|-------|
| CF Stack | `nautilus-dynamodb-stack` |
| Internal Table | `nautilus-cf-dynamodb-table` |

### Task Completion Checklist
- [x] ✅ CloudFormation stack `nautilus-dynamodb-stack` provisioned.
- [x] ✅ Stack creates the internal DynamoDB table.
- [x] ✅ `lifecycle` block correctly ignores `parameters` attribute.
- [x] ✅ All resources defined in `main.tf`.
- [x] ✅ `variables.tf` and `terraform.tfvars` used for table name.
- [x] ✅ `locals.tf` recognized for the template body.
- [x] ✅ `KKE_stack_name` exported in `outputs.tf`.
- [x] ✅ Final `terraform plan` returns **"No changes"**.
