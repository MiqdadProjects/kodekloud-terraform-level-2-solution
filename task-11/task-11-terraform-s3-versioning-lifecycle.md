# 🌟 Task 11 - Provision an S3 Bucket with Versioning and Lifecycle Configuration Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is implementing **lifecycle policies** to manage object storage efficiently in AWS. The goal is to create an S3 bucket with S3 Versioning enabled and a lifecycle rule that automatically **transitions objects to cheaper storage** after 30 days and **expires (deletes) them** after 365 days — reducing storage costs while preserving data for a meaningful retention window.

**Requirements:**
- Create an **S3 bucket** named **`devops-lifecycle-21055`**
- Enable **S3 Versioning** on the bucket
- Add a **lifecycle rule** named **`devops-lifecycle-rule`** with:
  - Transition to **`STANDARD_IA`** storage class after **30 days**
  - **Expiration** (deletion) of objects after **365 days**
- All resources must be defined in a single **`main.tf`** file (no separate `.tf` file)
- Use **`outputs.tf`** with output variable `KKE_bucket_name` — the created bucket name
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Build an automated S3 storage tiering and retention pipeline using Terraform — configuring versioning and lifecycle rules that reduce costs by moving infrequently accessed objects to cheaper storage classes and automatically expiring them.

💡 **Note:** S3 Versioning must be enabled before configuring lifecycle rules in Terraform. The `depends_on = [aws_s3_bucket_versioning.lifecycle_versioning]` on the lifecycle configuration resource enforces this ordering — ensuring AWS processes the versioning enablement before applying lifecycle policies.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- S3 bucket (`devops-lifecycle-21055`) — the storage target
- S3 bucket versioning — enabled on the bucket
- S3 lifecycle configuration — `STANDARD_IA` transition at day 30, expiration at day 365

**Working Directory:** `/home/bob/terraform`

**Lifecycle Rule Timeline:**
```
Day 0          Day 30              Day 365
  │              │                    │
  │ Object       │ Transition to      │ Object
  │ uploaded     │ STANDARD_IA        │ deleted
  │              │ (cheaper storage)  │ (auto-expiration)
  ▼              ▼                    ▼
[STANDARD] ──► [STANDARD_IA] ──────► [EXPIRED/DELETED]
 ~$0.023/GB     ~$0.0125/GB          No charge
```

**Storage Class Cost Comparison:**
| Storage Class | Cost/GB/Month | Use Case |
|---------------|--------------|----------|
| `STANDARD` | ~$0.023 | Frequently accessed data |
| `STANDARD_IA` | ~$0.0125 | Infrequent access (>30 days old) |
| *(Expired)* | $0 | Data no longer needed |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_s3_bucket`:** Creates the base bucket
- **`aws_s3_bucket_versioning`:** Enables versioning on the bucket (required before lifecycle rules in AWS provider v4+)
- **`aws_s3_bucket_lifecycle_configuration`:** Defines the `devops-lifecycle-rule` with `transition` and `expiration` blocks; `depends_on` ensures versioning is applied first

### 🎯 Implementation Strategy
1. Create `main.tf` with all three resources (bucket + versioning + lifecycle)
2. Create `outputs.tf` to export the bucket name as `KKE_bucket_name`
3. Initialize, validate, format, plan, apply, and verify

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf      # S3 bucket + versioning + lifecycle configuration
└── outputs.tf   # Output: KKE_bucket_name
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

### Step 2: Create Main Configuration File

Create `main.tf` with the S3 bucket, versioning, and lifecycle configuration:

```bash
cat > main.tf << 'EOF'
resource "aws_s3_bucket" "lifecycle_bucket" {
  bucket = "devops-lifecycle-21055"
}

# Enable Versioning
resource "aws_s3_bucket_versioning" "lifecycle_versioning" {
  bucket = aws_s3_bucket.lifecycle_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}

# Lifecycle Rule
resource "aws_s3_bucket_lifecycle_configuration" "lifecycle_rule" {
  bucket = aws_s3_bucket.lifecycle_bucket.id

  rule {
    id     = "devops-lifecycle-rule"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    expiration {
      days = 365
    }
  }

  depends_on = [
    aws_s3_bucket_versioning.lifecycle_versioning
  ]
}
EOF
```

**Purpose:** Create the complete S3 storage management stack — bucket, versioning, and lifecycle policy — in a single file.

**Configuration Breakdown:**

**Resource 1 — S3 Bucket (`aws_s3_bucket.lifecycle_bucket`):**
- `bucket = "devops-lifecycle-21055"` — Required bucket name (hardcoded, no variables needed for this task)

**Resource 2 — S3 Versioning (`aws_s3_bucket_versioning.lifecycle_versioning`):**
- `bucket = aws_s3_bucket.lifecycle_bucket.id` — Targets the created bucket (implicit dependency)
- `versioning_configuration.status = "Enabled"` — Activates S3 Versioning, allowing multiple versions of an object to be stored and recovered
- **Why enable versioning?** With versioning enabled, lifecycle expiration rules expire individual object versions — providing a safer data lifecycle pipeline

**Resource 3 — Lifecycle Configuration (`aws_s3_bucket_lifecycle_configuration.lifecycle_rule`):**
- `bucket = aws_s3_bucket.lifecycle_bucket.id` — Applies to the same bucket
- `rule.id = "devops-lifecycle-rule"` — Rule name as required
- `rule.status = "Enabled"` — Rule is active
- `transition.days = 30`, `transition.storage_class = "STANDARD_IA"` — After 30 days, move objects to Infrequent Access storage
- `expiration.days = 365` — After 365 days, delete the objects
- `depends_on = [aws_s3_bucket_versioning.lifecycle_versioning]` — **Explicit dependency:** ensures versioning is enabled **before** AWS processes the lifecycle configuration

**Transition vs Expiration:**
```hcl
transition {
  days          = 30
  storage_class = "STANDARD_IA"  # Move to cheaper storage
}

expiration {
  days = 365  # Permanently delete
}
```

**File Content (`main.tf`):**
```hcl
resource "aws_s3_bucket" "lifecycle_bucket" {
  bucket = "devops-lifecycle-21055"
}

# Enable Versioning
resource "aws_s3_bucket_versioning" "lifecycle_versioning" {
  bucket = aws_s3_bucket.lifecycle_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}

# Lifecycle Rule
resource "aws_s3_bucket_lifecycle_configuration" "lifecycle_rule" {
  bucket = aws_s3_bucket.lifecycle_bucket.id

  rule {
    id     = "devops-lifecycle-rule"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    expiration {
      days = 365
    }
  }

  depends_on = [
    aws_s3_bucket_versioning.lifecycle_versioning
  ]
}
```

---

### Step 3: Create Outputs Configuration File

Create `outputs.tf` to export the bucket name:

```bash
cat > outputs.tf << 'EOF'
output "KKE_bucket_name" {
  value = aws_s3_bucket.lifecycle_bucket.bucket
}
EOF
```

**File Content (`outputs.tf`):**
```hcl
output "KKE_bucket_name" {
  value = aws_s3_bucket.lifecycle_bucket.bucket
}
```

> 📝 **Note:** The output variable name is `KKE_bucket_name` (lowercase `b`) — exactly as specified in the task requirements.

---

### Step 4: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Expected Output:**
```
total 12
drwxr-xr-x 2 bob bob 4096 Feb 25 12:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 11:55 ..
-rw-r--r-- 1 bob bob  561 Feb 25 12:00 main.tf
-rw-r--r-- 1 bob bob   92 Feb 25 12:00 outputs.tf
```

**Success Indicators:**
- ✅ Both files present: `main.tf` and `outputs.tf`
- ✅ No extra `.tf` files created

---

### Step 5: Initialize Terraform

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

### Step 6: Validate Configuration

```bash
terraform validate
```

**Expected Output:**
```
Success! The configuration is valid.
```

**What's Validated:**
- ✅ `aws_s3_bucket_versioning` block is syntactically correct
- ✅ `aws_s3_bucket_lifecycle_configuration` rule block with `transition` and `expiration` is valid
- ✅ `depends_on` references the correct resource
- ✅ Output references the correct bucket attribute

---

### Step 7: Format Configuration Files

```bash
terraform fmt
```

**Expected Output:**
```
main.tf
outputs.tf
```

---

### Step 8: Review Execution Plan

```bash
terraform plan
```

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.lifecycle_bucket will be created
  + resource "aws_s3_bucket" "lifecycle_bucket" {
      + arn    = (known after apply)
      + bucket = "devops-lifecycle-21055"
      + id     = (known after apply)
      + region = (known after apply)
    }

  # aws_s3_bucket_lifecycle_configuration.lifecycle_rule will be created
  + resource "aws_s3_bucket_lifecycle_configuration" "lifecycle_rule" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + rule {
          + id     = "devops-lifecycle-rule"
          + status = "Enabled"

          + expiration {
              + days = 365
            }

          + transition {
              + days          = 30
              + storage_class = "STANDARD_IA"
            }
        }
    }

  # aws_s3_bucket_versioning.lifecycle_versioning will be created
  + resource "aws_s3_bucket_versioning" "lifecycle_versioning" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + versioning_configuration {
          + mfa_delete = (known after apply)
          + status     = "Enabled"
        }
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + KKE_bucket_name = "devops-lifecycle-21055"

───────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
```

**Plan Summary:**
- **Resources to Create:** 3 (bucket + versioning + lifecycle config)
- **Output:** `KKE_bucket_name = "devops-lifecycle-21055"` shown at plan time

**Success Indicators:**
- ✅ Lifecycle rule `devops-lifecycle-rule` shown with correct `transition` (30 days, `STANDARD_IA`) and `expiration` (365 days)
- ✅ Versioning `status = "Enabled"`
- ✅ `KKE_bucket_name` output pre-populated

---

### Step 9: Apply Configuration

```bash
terraform apply -auto-approve
```

**Expected Output:**
```
Plan: 3 to add, 0 to change, 0 to destroy.

aws_s3_bucket.lifecycle_bucket: Creating...
aws_s3_bucket.lifecycle_bucket: Creation complete after 2s [id=devops-lifecycle-21055]
aws_s3_bucket_versioning.lifecycle_versioning: Creating...
aws_s3_bucket_versioning.lifecycle_versioning: Creation complete after 1s [id=devops-lifecycle-21055]
aws_s3_bucket_lifecycle_configuration.lifecycle_rule: Creating...
aws_s3_bucket_lifecycle_configuration.lifecycle_rule: Creation complete after 1s [id=devops-lifecycle-21055]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

KKE_bucket_name = "devops-lifecycle-21055"
```

**Creation Order (enforced by `depends_on`):**
| Order | Resource | Reason |
|-------|----------|--------|
| 1st | `aws_s3_bucket.lifecycle_bucket` | Base bucket (no dependencies) |
| 2nd | `aws_s3_bucket_versioning.lifecycle_versioning` | Depends on bucket ID |
| 3rd | `aws_s3_bucket_lifecycle_configuration.lifecycle_rule` | Depends on versioning (explicit `depends_on`) |

**Success Indicators:**
- ✅ Bucket created first, versioning second, lifecycle last
- ✅ `KKE_bucket_name = "devops-lifecycle-21055"` output shown
- ✅ `3 added, 0 changed, 0 destroyed`

---

### Step 10: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Expected Output:**
```
aws_s3_bucket.lifecycle_bucket: Refreshing state... [id=devops-lifecycle-21055]
aws_s3_bucket_versioning.lifecycle_versioning: Refreshing state... [id=devops-lifecycle-21055]
aws_s3_bucket_lifecycle_configuration.lifecycle_rule: Refreshing state... [id=devops-lifecycle-21055]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**⚠️ Required before task submission.**

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Expected Output:**
```
aws_s3_bucket.lifecycle_bucket
aws_s3_bucket_lifecycle_configuration.lifecycle_rule
aws_s3_bucket_versioning.lifecycle_versioning
```

---

### Step 2: Inspect S3 Bucket

```bash
terraform state show aws_s3_bucket.lifecycle_bucket
```

**Expected Output:**
```
# aws_s3_bucket.lifecycle_bucket:
resource "aws_s3_bucket" "lifecycle_bucket" {
    arn    = "arn:aws:s3:::devops-lifecycle-21055"
    bucket = "devops-lifecycle-21055"
    id     = "devops-lifecycle-21055"
    region = "us-east-1"
}
```

---

### Step 3: Inspect Versioning Configuration

```bash
terraform state show aws_s3_bucket_versioning.lifecycle_versioning
```

**Expected Output:**
```
# aws_s3_bucket_versioning.lifecycle_versioning:
resource "aws_s3_bucket_versioning" "lifecycle_versioning" {
    bucket = "devops-lifecycle-21055"
    id     = "devops-lifecycle-21055"

    versioning_configuration {
        status = "Enabled"
    }
}
```

**Verification Points:**
- ✅ `status = "Enabled"` — versioning is active

---

### Step 4: Inspect Lifecycle Configuration

```bash
terraform state show aws_s3_bucket_lifecycle_configuration.lifecycle_rule
```

**Expected Output:**
```
# aws_s3_bucket_lifecycle_configuration.lifecycle_rule:
resource "aws_s3_bucket_lifecycle_configuration" "lifecycle_rule" {
    bucket = "devops-lifecycle-21055"
    id     = "devops-lifecycle-21055"

    rule {
        id     = "devops-lifecycle-rule"
        status = "Enabled"

        expiration {
            days = 365
        }

        transition {
            days          = 30
            storage_class = "STANDARD_IA"
        }
    }
}
```

**Verification Points:**
- ✅ `id = "devops-lifecycle-rule"` — correct rule name
- ✅ `transition.days = 30`, `transition.storage_class = "STANDARD_IA"`
- ✅ `expiration.days = 365`

---

### Step 5: View Output Value

```bash
terraform output
```

**Expected Output:**
```
KKE_bucket_name = "devops-lifecycle-21055"
```

---

### Step 6: AWS CLI Verification (Optional)

```bash
# Verify versioning is enabled on the bucket
aws s3api get-bucket-versioning \
  --bucket devops-lifecycle-21055 \
  --query "Status" \
  --output text

# Verify lifecycle rules are configured
aws s3api get-bucket-lifecycle-configuration \
  --bucket devops-lifecycle-21055 \
  --query "Rules[].{ID:ID,Status:Status,Transition:Transitions,Expiration:Expiration}" \
  --output table
```

**Expected Output:**
```
Enabled
```

---

## 🔍 Code Analysis

### Resource Dependency Chain

```
aws_s3_bucket.lifecycle_bucket
        │ .id → bucket ID
        ▼
aws_s3_bucket_versioning.lifecycle_versioning
        │ (must complete before lifecycle rules)
        ▼  explicit depends_on
aws_s3_bucket_lifecycle_configuration.lifecycle_rule
        │
        └── rule: devops-lifecycle-rule
            │  transition  → day 30  → STANDARD_IA
            └── expiration → day 365 → DELETED
```

### Why `depends_on` Is Required for the Lifecycle Configuration

```hcl
# Without depends_on — risk of race condition:
# AWS may reject lifecycle rules applied before versioning is fully processed
resource "aws_s3_bucket_lifecycle_configuration" "lifecycle_rule" {
  # ...
}

# With depends_on — safe ordering guaranteed:
resource "aws_s3_bucket_lifecycle_configuration" "lifecycle_rule" {
  # ...
  depends_on = [aws_s3_bucket_versioning.lifecycle_versioning]
}
```

Both `aws_s3_bucket_versioning` and `aws_s3_bucket_lifecycle_configuration` only use the bucket `id` (same value), so Terraform cannot infer a dependency between them from attribute references alone. The explicit `depends_on` is necessary to guarantee the correct ordering.

### Lifecycle Rule Deep Dive

```hcl
rule {
  id     = "devops-lifecycle-rule"   # Rule identifier
  status = "Enabled"                 # Rule is active

  transition {
    days          = 30               # Days since object creation
    storage_class = "STANDARD_IA"   # Target storage class
  }

  expiration {
    days = 365                       # Days until object is permanently deleted
  }
}
```

**Key behaviors:**
| Configuration | Behavior |
|---------------|----------|
| `transition.days = 30` | Objects last modified >30 days ago move to `STANDARD_IA` |
| `transition.storage_class = "STANDARD_IA"` | Infrequent Access — 45-day minimum storage, retrieval fee |
| `expiration.days = 365` | Objects (and all their versions with versioning enabled) are deleted after 365 days |

### S3 Storage Classes Available for Lifecycle Transitions

| Storage Class | Min Storage Duration | Use Case |
|---------------|---------------------|----------|
| `STANDARD_IA` | 30 days | Infrequent access, rapid retrieval |
| `ONEZONE_IA` | 30 days | Non-critical, infrequent access |
| `INTELLIGENT_TIERING` | None | Unknown access patterns |
| `GLACIER_IR` | 90 days | Archive, millisecond retrieval |
| `GLACIER` | 90 days | Archive, minutes/hours retrieval |
| `DEEP_ARCHIVE` | 180 days | Long-term archive, 12-hour retrieval |

### Versioning Interaction with Lifecycle Rules

| Versioning State | Lifecycle `expiration` Behavior |
|-----------------|--------------------------------|
| **Disabled** | Permanently deletes the current object |
| **Enabled** | Creates a delete marker (object becomes non-current); all versions are managed independently |
| **Suspended** | Same as disabled for new objects |

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate configuration
terraform validate

# 2. Preview plan
terraform plan

# 3. Apply
terraform apply -auto-approve

# 4. Verify all 3 resources in state
terraform state list

# 5. Check versioning enabled
terraform state show aws_s3_bucket_versioning.lifecycle_versioning

# 6. Check lifecycle rule details
terraform state show aws_s3_bucket_lifecycle_configuration.lifecycle_rule

# 7. View outputs
terraform output

# 8. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| Bucket name | `devops-lifecycle-21055` | `terraform output KKE_bucket_name` |
| Versioning | `Enabled` | `terraform state show aws_s3_bucket_versioning.lifecycle_versioning` |
| Lifecycle rule ID | `devops-lifecycle-rule` | `terraform state show aws_s3_bucket_lifecycle_configuration.lifecycle_rule` |
| Transition days | `30` | `terraform state show aws_s3_bucket_lifecycle_configuration.lifecycle_rule` |
| Transition class | `STANDARD_IA` | `terraform state show aws_s3_bucket_lifecycle_configuration.lifecycle_rule` |
| Expiration days | `365` | `terraform state show aws_s3_bucket_lifecycle_configuration.lifecycle_rule` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: Lifecycle Configuration Rejected Before Versioning

**Error:**
```
Error: putting S3 Bucket Lifecycle Configuration: InvalidArgument:
```

**Cause:** The lifecycle configuration was applied before versioning was fully processed by AWS. This can happen if `depends_on` is missing.

**Solution:** Ensure `main.tf` has the `depends_on` in the lifecycle resource:
```hcl
resource "aws_s3_bucket_lifecycle_configuration" "lifecycle_rule" {
  # ...
  depends_on = [aws_s3_bucket_versioning.lifecycle_versioning]
}
```

Then re-apply:
```bash
terraform apply -auto-approve
```

---

### Issue 2: Bucket Name Already Exists

**Error:**
```
Error: creating S3 Bucket: BucketAlreadyOwnedByYou: Your previous request to create the named bucket succeeded and you already own it.
```

**Solution:**
```bash
# Import existing bucket into state
terraform import aws_s3_bucket.lifecycle_bucket devops-lifecycle-21055

# Apply to configure versioning and lifecycle rule on the existing bucket
terraform apply -auto-approve
```

---

### Issue 3: Plan Shows Lifecycle Rule Changes After Apply

**Symptom:**
```
Plan: 0 to add, 1 to change, 0 to destroy.
~ aws_s3_bucket_lifecycle_configuration.lifecycle_rule
  ~ rule.filter = {} → null
```

**Cause:** AWS provider v4+ may add a default empty `filter {}` block to lifecycle rules. This causes Terraform to detect configuration drift.

**Solution:** Explicitly add an empty filter block to match what AWS applies:
```hcl
rule {
  id     = "devops-lifecycle-rule"
  status = "Enabled"

  filter {}   # Add this to match AWS default

  transition {
    days          = 30
    storage_class = "STANDARD_IA"
  }

  expiration {
    days = 365
  }
}
```

Then re-apply:
```bash
terraform apply -auto-approve
terraform plan   # Should show "No changes"
```

---

### Issue 4: `STANDARD_IA` Minimum Storage Duration Charge

**Symptom:** Objects deleted or transitioned within 30 days still incur `STANDARD_IA` charges.

**Cause:** `STANDARD_IA` has a **30-day minimum storage duration**. Objects deleted before 30 days are still charged for the full 30 days.

**Solution:** Set the `transition.days` to at least `30`:
```hcl
transition {
  days = 30   # ✅ Minimum to avoid extra charges
}
```

---

### Issue 5: Versioning Cannot Be Disabled After Enabling

**Warning:** Once S3 Versioning is enabled on a bucket, it can only be **suspended** — not fully disabled.

**Impact:** Changing `status = "Enabled"` to `status = "Suspended"` in Terraform is valid, but you cannot revert to a "never versioned" state.

**Production Consideration:** Before enabling versioning in production, confirm this is acceptable for the bucket's lifecycle — and budget for the additional storage costs of keeping multiple object versions.

---

## 📚 Best Practices

### 1. Always Enable Versioning Before Applying Lifecycle Rules
Use `depends_on` to ensure Terraform creates versioning before the lifecycle configuration. AWS may reject or mis-apply lifecycle rules on buckets where versioning state is still being processed.

### 2. Use Lifecycle Rules for Automatic Cost Optimization
For buckets storing logs, backups, or telemetry data:
- **Transition at day 30-90** → `STANDARD_IA` or `INTELLIGENT_TIERING`
- **Transition at day 90-180** → `GLACIER` for archival
- **Expire at day 365+** → permanent deletion

### 3. Add Prefix Filters for Targeted Rules
Apply lifecycle rules to specific paths within a bucket rather than the entire bucket:
```hcl
rule {
  id = "devops-lifecycle-rule"

  filter {
    prefix = "logs/"   # Only apply to objects under the "logs/" prefix
  }

  transition { ... }
  expiration { ... }
}
```

### 4. Monitor Lifecycle Transitions via S3 Storage Lens
Use AWS S3 Storage Lens or S3 Inventory to confirm objects are being transitioned and expired as expected — lifecycle rules take 24 hours to process after cutover.

### 5. Track Non-Current Version Expiration for Versioned Buckets
With versioning enabled, when an object is "deleted" a delete marker is created. To fully clean up old versions, add a `noncurrent_version_expiration` block:
```hcl
noncurrent_version_expiration {
  noncurrent_days = 90   # Delete non-current versions after 90 days
}
```

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Name | Configuration |
|----------|------|---------------|
| S3 Bucket | `devops-lifecycle-21055` | Base storage bucket |
| S3 Versioning | `devops-lifecycle-21055` | `status = "Enabled"` |
| Lifecycle Rule | `devops-lifecycle-rule` | `STANDARD_IA` at 30 days, expire at 365 days |

### Files Created

| File | Purpose |
|------|---------|
| `main.tf` | S3 bucket + versioning + lifecycle configuration (all in one file) |
| `outputs.tf` | Exports `KKE_bucket_name` |

### Task Completion Checklist

- [x] ✅ S3 bucket `devops-lifecycle-21055` created
- [x] ✅ S3 Versioning enabled on the bucket (`status = "Enabled"`)
- [x] ✅ Lifecycle rule `devops-lifecycle-rule` configured with `status = "Enabled"`
- [x] ✅ Transition to `STANDARD_IA` after **30 days**
- [x] ✅ Expiration (deletion) after **365 days**
- [x] ✅ `depends_on` ensures versioning is applied before lifecycle configuration
- [x] ✅ All resources defined in **single `main.tf`** file
- [x] ✅ `outputs.tf` exports `KKE_bucket_name = "devops-lifecycle-21055"`
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply` — **3 resources added**
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task demonstrates **S3 lifecycle management** — one of the most impactful cost optimization techniques in AWS. The three-resource Terraform chain (`bucket` → `versioning` → `lifecycle`) must be provisioned in strict order, which is enforced by the explicit `depends_on`. The lifecycle rule creates a fully automated **tiered storage pipeline**: objects start in `STANDARD`, move to `STANDARD_IA` at day 30 (cutting costs by ~46%), and are automatically deleted at day 365 — with no manual intervention required.
