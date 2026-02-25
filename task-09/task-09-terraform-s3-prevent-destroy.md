# 🌟 Task 09 - Provision an S3 Bucket with `prevent_destroy` Lifecycle Protection Using Terraform

## 📌 Task Description

The **DevOps team** must configure an S3 bucket with **strict lifecycle protections** to guard against accidental deletion. By applying the `prevent_destroy` lifecycle meta-argument, Terraform will refuse to execute any plan that would result in the bucket being destroyed — providing a critical safety net for production storage resources.

**Requirements:**
- Create an **S3 bucket** named **`datacenter-s3-7329`**
- Apply the **`prevent_destroy`** lifecycle rule to protect the bucket from accidental destruction
- All resources must be defined in a single **`main.tf`** file
- Use **`variables.tf`** with:
  - `KKE_BUCKET_NAME` — bucket name variable (no default; value provided via tfvars)
- Use **`terraform.tfvars`** to set the bucket name value
- Use **`outputs.tf`** with:
  - `s3_bucket_name` — name of the created bucket
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Provision a protected S3 bucket via Terraform where the `prevent_destroy` lifecycle rule ensures Terraform **cannot destroy it by mistake**, regardless of what commands are run.

💡 **Note:** `prevent_destroy = true` is a **Terraform-level guard** — it does not affect the AWS resource itself. It works by causing `terraform destroy` (or any plan that would delete the resource) to fail with an error, forcing the operator to explicitly remove the lifecycle block before any deletion can proceed.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- AWS S3 bucket (`datacenter-s3-7329`) with `prevent_destroy` lifecycle protection

**Working Directory:** `/home/bob/terraform`

**Configuration Summary:**
| Resource | Name | Protection |
|----------|------|-----------|
| S3 Bucket | `datacenter-s3-7329` | `lifecycle { prevent_destroy = true }` |

**Why `prevent_destroy`?**
| Scenario | Without `prevent_destroy` | With `prevent_destroy` |
|----------|--------------------------|----------------------|
| `terraform destroy` | ✅ Bucket deleted | ❌ Error — destroy blocked |
| `terraform apply` removes bucket | ✅ Bucket deleted | ❌ Error — destroy blocked |
| Normal `terraform apply` | ✅ Works normally | ✅ Works normally |
| `terraform plan` | ✅ Works normally | ✅ Works normally |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_s3_bucket`:** Creates the S3 bucket using the `KKE_BUCKET_NAME` variable
- **`lifecycle { prevent_destroy = true }`:** Embedded lifecycle block that instructs Terraform to reject any plan that would destroy this resource
- **`variable "KKE_BUCKET_NAME"`:** No default — value must be supplied via `terraform.tfvars`
- **`terraform.tfvars`:** Supplies `KKE_BUCKET_NAME = "datacenter-s3-7329"` at runtime

### 🎯 Implementation Strategy
1. Create `variables.tf` with `KKE_BUCKET_NAME` (no default)
2. Create `terraform.tfvars` with the actual bucket name
3. Create `main.tf` with the S3 bucket + `prevent_destroy` lifecycle block
4. Create `outputs.tf` to export the bucket name
5. Initialize, validate, format, plan, apply, and verify

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # S3 bucket with prevent_destroy lifecycle
├── variables.tf       # KKE_BUCKET_NAME variable declaration
├── terraform.tfvars   # KKE_BUCKET_NAME = "datacenter-s3-7329"
└── outputs.tf         # Output: bucket name
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

**Verification:**
```bash
pwd
```

**Expected Output:**
```
/home/bob/terraform
```

---

### Step 2: Create Variables Definition File

Create `variables.tf` with the `KKE_BUCKET_NAME` variable:

```bash
cat > variables.tf << 'EOF'
variable "KKE_BUCKET_NAME" {
  description = "Name of the S3 bucket"
  type        = string
}
EOF
```

**Purpose:** Declare the bucket name variable **without a default** — this enforces that the value must be explicitly provided, preventing accidental bucket creation with a wrong or missing name.

**Configuration Breakdown:**
- `variable "KKE_BUCKET_NAME"` — Variable name as required by the task
- `type = string` — Enforces string type
- No `default` — Value **must** be supplied via `terraform.tfvars` or `-var` flag; Terraform will prompt interactively if neither is provided

**File Content (`variables.tf`):**
```hcl
variable "KKE_BUCKET_NAME" {
  description = "Name of the S3 bucket"
  type        = string
}
```

---

### Step 3: Create Terraform Variables File

Create `terraform.tfvars` to supply the bucket name value:

```bash
cat > terraform.tfvars << 'EOF'
KKE_BUCKET_NAME = "datacenter-s3-7329"
EOF
```

**Purpose:** Provide the actual bucket name value separately from the variable declaration — following Terraform's recommended pattern of keeping variable definitions and values in separate files.

**Configuration Breakdown:**
- `KKE_BUCKET_NAME = "datacenter-s3-7329"` — The required bucket name
- Terraform automatically loads `terraform.tfvars` at every `plan` and `apply` — no `-var-file` flag required

**File Content (`terraform.tfvars`):**
```hcl
KKE_BUCKET_NAME = "datacenter-s3-7329"
```

---

### Step 4: Create Main Configuration File

Create `main.tf` with the S3 bucket and `prevent_destroy` lifecycle rule:

```bash
cat > main.tf << 'EOF'
resource "aws_s3_bucket" "protected_bucket" {
  bucket = var.KKE_BUCKET_NAME

  lifecycle {
    prevent_destroy = true
  }
}
EOF
```

**Purpose:** Provision the S3 bucket with a lifecycle block that prevents Terraform from ever destroying it.

**Configuration Breakdown:**
- `bucket = var.KKE_BUCKET_NAME` — Names the bucket using the variable (value from `terraform.tfvars`)
- `lifecycle { prevent_destroy = true }` — The core protection mechanism:
  - When Terraform generates a plan that would destroy this resource, it **immediately raises an error** and aborts
  - This applies to `terraform destroy`, removing the resource block from `main.tf`, or any refactoring that would cause recreation

**How `prevent_destroy` Works:**
```
terraform destroy
    │
    ▼
Terraform generates execution plan
    │
    ▼
Plan includes: "aws_s3_bucket.protected_bucket" → destroy
    │
    ▼
Terraform checks lifecycle rules
    │
    ▼
prevent_destroy = true → ERROR:
Error: Instance cannot be destroyed
│  on main.tf line 1, in resource "aws_s3_bucket" "protected_bucket":
│  │ resource "aws_s3_bucket" "protected_bucket" {
│  This object is protected from destroy action. To allow this object to
│  be destroyed, remove the lifecycle block.
```

**File Content (`main.tf`):**
```hcl
resource "aws_s3_bucket" "protected_bucket" {
  bucket = var.KKE_BUCKET_NAME

  lifecycle {
    prevent_destroy = true
  }
}
```

---

### Step 5: Create Outputs Configuration File

Create `outputs.tf` to export the bucket name:

```bash
cat > outputs.tf << 'EOF'
output "s3_bucket_name" {
  value = aws_s3_bucket.protected_bucket.bucket
}
EOF
```

**File Content (`outputs.tf`):**
```hcl
output "s3_bucket_name" {
  value = aws_s3_bucket.protected_bucket.bucket
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
drwxr-xr-x 2 bob bob 4096 Feb 25 11:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 10:55 ..
-rw-r--r-- 1 bob bob  118 Feb 25 11:00 main.tf
-rw-r--r-- 1 bob bob   86 Feb 25 11:00 outputs.tf
-rw-r--r-- 1 bob bob   36 Feb 25 11:00 terraform.tfvars
-rw-r--r-- 1 bob bob   92 Feb 25 11:00 variables.tf
```

**Success Indicators:**
- ✅ All four files present
- ✅ No extra `.tf` files created

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
- ✅ `lifecycle { prevent_destroy = true }` syntax is correct
- ✅ `var.KKE_BUCKET_NAME` reference is valid
- ✅ Output references resolve correctly

---

### Step 9: Format Configuration Files

```bash
terraform fmt
```

**Expected Output:**
```
main.tf
outputs.tf
variables.tf
```

---

### Step 10: Review Execution Plan

```bash
terraform plan
```

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.protected_bucket will be created
  + resource "aws_s3_bucket" "protected_bucket" {
      + arn                         = (known after apply)
      + bucket                      = "datacenter-s3-7329"
      + id                          = (known after apply)
      + region                      = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + s3_bucket_name = "datacenter-s3-7329"

───────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
```

> 📝 **Note:** The `lifecycle { prevent_destroy = true }` block is a Terraform meta-argument and is **not shown in the plan output** — it is enforced at plan generation time, not at apply time.

**Success Indicators:**
- ✅ Bucket named `datacenter-s3-7329` (from `terraform.tfvars`)
- ✅ Output `s3_bucket_name = "datacenter-s3-7329"` shown
- ✅ `1 to add, 0 to change, 0 to destroy`

---

### Step 11: Apply Configuration

```bash
terraform apply -auto-approve
```

**Expected Output:**
```
Plan: 1 to add, 0 to change, 0 to destroy.

aws_s3_bucket.protected_bucket: Creating...
aws_s3_bucket.protected_bucket: Creation complete after 2s [id=datacenter-s3-7329]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

s3_bucket_name = "datacenter-s3-7329"
```

**Success Indicators:**
- ✅ Bucket `datacenter-s3-7329` created
- ✅ Output `s3_bucket_name = "datacenter-s3-7329"` displayed
- ✅ `1 added, 0 changed, 0 destroyed`

---

### Step 12: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Expected Output:**
```
aws_s3_bucket.protected_bucket: Refreshing state... [id=datacenter-s3-7329]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ Task is ready for submission

---

### Step 13: Verify `prevent_destroy` Is Active (Optional Demonstration)

```bash
terraform destroy -auto-approve
```

**Expected Output (demonstrates protection working correctly):**
```
╷
│ Error: Instance cannot be destroyed
│
│   on main.tf line 1, in resource "aws_s3_bucket" "protected_bucket":
│    1: resource "aws_s3_bucket" "protected_bucket" {
│
│ This object is protected from destroy action. The "prevent_destroy"
│ lifecycle meta-argument is set to true. To allow this object to be
│ destroyed, remove the lifecycle block or set its prevent_destroy
│ argument to false.
╵
```

✅ **This error is expected and confirms the lifecycle protection is working correctly.**

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Expected Output:**
```
aws_s3_bucket.protected_bucket
```

---

### Step 2: Inspect Bucket Details

```bash
terraform state show aws_s3_bucket.protected_bucket
```

**Expected Output:**
```
# aws_s3_bucket.protected_bucket:
resource "aws_s3_bucket" "protected_bucket" {
    arn    = "arn:aws:s3:::datacenter-s3-7329"
    bucket = "datacenter-s3-7329"
    id     = "datacenter-s3-7329"
    region = "us-east-1"
}
```

**Verification Points:**
- ✅ `bucket = "datacenter-s3-7329"` — correct name from `terraform.tfvars`
- ✅ `id = "datacenter-s3-7329"` — bucket exists in AWS

---

### Step 3: View Output Values

```bash
terraform output
```

**Expected Output:**
```
s3_bucket_name = "datacenter-s3-7329"
```

---

### Step 4: AWS CLI Verification (Optional)

```bash
# Verify bucket exists
aws s3 ls | grep datacenter-s3-7329
```

**Expected Output:**
```
2024-02-25 11:00:00 datacenter-s3-7329
```

---

## 🔍 Code Analysis

### `prevent_destroy` Lifecycle Meta-Argument

```hcl
resource "aws_s3_bucket" "protected_bucket" {
  bucket = var.KKE_BUCKET_NAME

  lifecycle {
    prevent_destroy = true    # ← Terraform-level guard; not an AWS feature
  }
}
```

**How it's enforced:**
```
Any Terraform plan that would DESTROY aws_s3_bucket.protected_bucket
                │
                ▼
       Terraform raises an error immediately
       Plan is NOT generated
       No AWS API calls are made
```

### Variable → tfvars → Resource Flow

```
variables.tf                 terraform.tfvars           main.tf
────────────                 ────────────────           ───────
variable "KKE_BUCKET_NAME"   KKE_BUCKET_NAME =          bucket =
  type = string         →    "datacenter-s3-7329"   →   var.KKE_BUCKET_NAME
  (no default)                                          = "datacenter-s3-7329"
```

**Why no default in `variables.tf`?**  
Omitting the `default` value forces the bucket name to be explicitly declared in `terraform.tfvars`. This is intentional — for a protected production bucket, there should be no "accidental" default name. Every deployment must consciously specify the bucket name.

### Four Lifecycle Meta-Arguments Compared

| Meta-Argument | Purpose | This Task |
|--------------|---------|-----------|
| `prevent_destroy` | Block resource destruction | ✅ Used |
| `create_before_destroy` | Create replacement before destroying old | ❌ Not used |
| `ignore_changes` | Ignore specific attribute drifts | ❌ Not used |
| `replace_triggered_by` | Force replacement when other resources change | ❌ Not used |

### Variable Definition vs Value Assignment

| File | Role | Contains |
|------|------|---------|
| `variables.tf` | **Declaration** — defines the variable schema | `variable "KKE_BUCKET_NAME" { type = string }` |
| `terraform.tfvars` | **Assignment** — provides the actual value | `KKE_BUCKET_NAME = "datacenter-s3-7329"` |

Keeping these in separate files is a Terraform best practice — it enables the same variable declaration to be reused across different environments (dev/staging/prod) by swapping only the `.tfvars` file.

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate syntax
terraform validate

# 2. Preview plan
terraform plan

# 3. Apply
terraform apply -auto-approve

# 4. Verify in state
terraform state list
terraform state show aws_s3_bucket.protected_bucket

# 5. View output
terraform output

# 6. Optional: verify prevent_destroy is active
terraform destroy   # Should fail with lifecycle error

# 7. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| Bucket name | `datacenter-s3-7329` | `terraform output s3_bucket_name` |
| Bucket exists | In AWS S3 | `aws s3 ls \| grep datacenter-s3-7329` |
| `prevent_destroy` active | Error on destroy attempt | `terraform destroy` → expect error |
| Variable source | `terraform.tfvars` | `terraform console` → `var.KKE_BUCKET_NAME` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: Variable Not Found

**Error:**
```
│ Error: No value for required variable
│
│   on variables.tf line 1:
│    1: variable "KKE_BUCKET_NAME" {
│
│ The root module input variable "KKE_BUCKET_NAME" is not set, and has no default value.
```

**Cause:** `terraform.tfvars` is missing or the variable name in `.tfvars` doesn't match the declaration.

**Solution:**
```bash
# Verify terraform.tfvars exists and contains the correct variable name
cat terraform.tfvars
# Expected: KKE_BUCKET_NAME = "datacenter-s3-7329"

# Re-create if missing
echo 'KKE_BUCKET_NAME = "datacenter-s3-7329"' > terraform.tfvars
```

---

### Issue 2: Bucket Name Already Exists

**Error:**
```
Error: creating S3 Bucket: BucketAlreadyExists: The requested bucket name is not available.
```

**Cause:** S3 bucket names are **globally unique** across all AWS accounts. The name `datacenter-s3-7329` is already taken by another account.

**Solution:**
```bash
# Check if the bucket exists in YOUR account
aws s3 ls | grep datacenter-s3-7329

# If it's in your account, import it into state instead
terraform import aws_s3_bucket.protected_bucket datacenter-s3-7329

# Then run
terraform plan   # Should show "No changes"
```

---

### Issue 3: `prevent_destroy` Does Not Stop `terraform destroy`

**Symptom:** Running `terraform destroy` succeeds instead of failing with an error.

**Cause:** The `lifecycle { prevent_destroy = true }` block was accidentally removed from `main.tf` or there is a syntax error.

**Solution:**
```bash
# Verify the lifecycle block is present
cat main.tf | grep -A 3 "lifecycle"

# Expected output:
# lifecycle {
#   prevent_destroy = true
# }

# If missing, re-add and apply
terraform apply -auto-approve
```

---

### Issue 4: How to Intentionally Destroy a `prevent_destroy` Bucket

**Scenario:** You need to tear down the environment after task completion.

**Solution:** Remove the lifecycle block first, then destroy:

```hcl
# Step 1: Edit main.tf — remove or comment out the lifecycle block
resource "aws_s3_bucket" "protected_bucket" {
  bucket = var.KKE_BUCKET_NAME

  # lifecycle {
  #   prevent_destroy = true
  # }
}
```

```bash
# Step 2: Apply the configuration change (remove lifecycle protection)
terraform apply -auto-approve

# Step 3: Now destroy is allowed
terraform destroy -auto-approve
```

---

### Issue 5: Plan Shows Unexpected Changes After Apply

**Symptom:**
```
Plan: 0 to add, 1 to change, 0 to destroy.
~ aws_s3_bucket.protected_bucket
  ~ tags_all = {}  → { "CreatedBy" = "terraform" }
```

**Cause:** AWS account-level tag policies or default tags may be automatically applied.

**Solution:**
```bash
# Force state refresh to sync with actual AWS state
terraform refresh

# Then check the plan
terraform plan   # Should show "No changes" after refresh
```

---

## 📚 Best Practices

### 1. Always Apply `prevent_destroy` to Critical Production Buckets
S3 buckets containing application data, backups, or audit logs should always have `prevent_destroy = true`. One accidental `terraform destroy` can cause irreversible data loss.

### 2. Separate Variable Declaration from Values
Using `variables.tf` (declaration) + `terraform.tfvars` (values) enables:
- **Environment flexibility:** Swap `.tfvars` files for dev/staging/prod
- **Security:** `.tfvars` files with secrets can be excluded from version control via `.gitignore`
- **Clarity:** Variable schemas are documented independently from runtime values

### 3. Never Use Default Values for Protected Resource Names
Omitting `default` in `variables.tf` for bucket names forces explicit intent — no one accidentally deploys a protected bucket with a wrong name:
```hcl
# ✅ No default — value must be explicitly supplied
variable "KKE_BUCKET_NAME" {
  type = string
}

# ⚠️ Default allows accidental deployment with wrong name
variable "KKE_BUCKET_NAME" {
  type    = string
  default = "accidental-name-here"
}
```

### 4. Combine `prevent_destroy` with S3 Versioning for Full Protection
`prevent_destroy` protects the bucket from Terraform deletion. S3 Versioning protects individual objects from accidental overwrites or deletions:
```hcl
resource "aws_s3_bucket_versioning" "protected_bucket_versioning" {
  bucket = aws_s3_bucket.protected_bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

### 5. Document the Intentional Removal Process
If `prevent_destroy` is set, document in your team's runbook how to safely remove it when decommissioning the bucket — preventing future confusion.

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Name | Protection |
|----------|------|-----------|
| S3 Bucket | `datacenter-s3-7329` | `lifecycle { prevent_destroy = true }` |

### Files Created

| File | Purpose |
|------|---------|
| `main.tf` | S3 bucket with `prevent_destroy` lifecycle block |
| `variables.tf` | `KKE_BUCKET_NAME` variable (no default) |
| `terraform.tfvars` | `KKE_BUCKET_NAME = "datacenter-s3-7329"` |
| `outputs.tf` | Exports `s3_bucket_name` |

### Task Completion Checklist

- [x] ✅ S3 bucket `datacenter-s3-7329` created
- [x] ✅ `lifecycle { prevent_destroy = true }` applied — bucket is destroy-protected
- [x] ✅ All resources defined in **single `main.tf`** file
- [x] ✅ `variables.tf` declares `KKE_BUCKET_NAME` without a default value
- [x] ✅ `terraform.tfvars` supplies `KKE_BUCKET_NAME = "datacenter-s3-7329"`
- [x] ✅ `outputs.tf` exports `s3_bucket_name`
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply` — **1 resource added**
- [x] ✅ `terraform destroy` → **Error: Instance cannot be destroyed** (protection confirmed)
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task demonstrates the `prevent_destroy` lifecycle meta-argument — Terraform's built-in safeguard for critical infrastructure resources. Unlike AWS resource policies or S3 bucket policies, `prevent_destroy` operates entirely at the **Terraform plan layer**: it intercepts and rejects any execution plan that would destroy the protected resource, regardless of who initiated the destroy. Using it alongside variable-driven bucket naming and `terraform.tfvars` for value injection represents a clean, production-ready Terraform configuration pattern for protecting sensitive storage infrastructure.
