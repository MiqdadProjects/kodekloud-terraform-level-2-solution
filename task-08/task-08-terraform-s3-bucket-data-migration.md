# 🌟 Task 08 - S3 Bucket Creation and Data Migration Using Terraform

## 📌 Task Description

As part of a **data migration project**, the team needs to migrate all data from an existing S3 bucket to a newly provisioned private S3 bucket. The migration must be complete, accurate, and verified — with both buckets containing identical data after the process. Terraform is used to provision the new bucket and orchestrate the data sync using a `null_resource` provisioner that invokes `aws s3 sync` via a local-exec command.

**Requirements:**
- Create a new **private S3 bucket** named **`devops-sync-2251`**, stored in a variable named `KKE_BUCKET`
- Migrate all data from the existing **`devops-s3-4515`** bucket to the new bucket
- Ensure **both buckets contain the same data** after migration
- Update **`main.tf`** (do not create a separate `.tf` file) to provision the bucket, set ACL, and run the sync
- Use **`variables.tf`** with:
  - `KKE_BUCKET` — name of the new bucket
- Use **`outputs.tf`** with:
  - `new_kke_bucket_name` — name of the new bucket
  - `new_kke_bucket_acl` — ACL of the new bucket
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Provision a private S3 bucket and sync all objects from a source bucket using Terraform's `null_resource` with a `local-exec` provisioner — implementing Infrastructure-as-Code-driven data migration.

💡 **Note:** The `null_resource` with a `local-exec` provisioner is the standard Terraform pattern for running shell commands as part of an infrastructure deployment. In this task, it drives the `aws s3 sync` command that migrates data between buckets. The `--endpoint-url` flag is used because this runs on a LocalStack environment (`http://aws:4566`).

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS / LocalStack (`http://aws:4566`)  
**Provider:** AWS (Amazon Web Services / LocalStack)  
**Resources:**
- New private S3 bucket (`devops-sync-2251`)
- S3 bucket ACL — set to `private`
- `null_resource` with `local-exec` provisioner to run `aws s3 sync`

**Working Directory:** `/home/bob/terraform`

**Migration Overview:**
| | Source Bucket | Destination Bucket |
|-|------|------|
| **Name** | `devops-s3-4515` | `devops-sync-2251` |
| **Status** | Pre-existing | Created by Terraform |
| **ACL** | Existing | `private` |
| **Data** | Original data | Synced copy |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_s3_bucket`:** Creates the new private S3 bucket using the `KKE_BUCKET` variable
- **`aws_s3_bucket_acl`:** Sets the bucket's access control to `private`
- **`null_resource` + `local-exec`:** Runs `aws s3 sync` to copy all objects from the source to destination bucket; `depends_on` ensures bucket + ACL are fully ready before sync begins
- **Variable (`KKE_BUCKET`):** Default value `"devops-sync-2251"` makes the bucket name configurable
- **Outputs:** Exposes bucket name and ACL for verification

### 🎯 Implementation Strategy
1. Create `variables.tf` with `KKE_BUCKET` variable and default value
2. Update `main.tf` with S3 bucket, ACL resource, and sync `null_resource`
3. Create `outputs.tf` to export bucket name and ACL
4. Initialize, validate, format, plan, apply, and verify

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf        # S3 bucket + ACL + null_resource sync
├── variables.tf   # KKE_BUCKET variable declaration
└── outputs.tf     # Outputs: bucket name + ACL
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

**Purpose:** Ensure all Terraform commands execute in the correct directory.

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

Create `variables.tf` with the `KKE_BUCKET` variable:

```bash
cat > variables.tf << 'EOF'
variable "KKE_BUCKET" {
  description = "Name of the new S3 bucket"
  type        = string
  default     = "devops-sync-2251"
}
EOF
```

**Purpose:** Define the new bucket name as a configurable variable with a default value baked in — no `terraform.tfvars` required.

**Configuration Breakdown:**
- `variable "KKE_BUCKET"` — Declares the bucket name variable
- `type = string` — Enforces string type
- `default = "devops-sync-2251"` — Sets the required bucket name as the default; no extra value assignment needed
- **Best Practice:** Using `default` for environment-specific names makes overriding via `-var` flags simple at plan/apply time

**File Content (`variables.tf`):**
```hcl
variable "KKE_BUCKET" {
  description = "Name of the new S3 bucket"
  type        = string
  default     = "devops-sync-2251"
}
```

---

### Step 3: Update Main Configuration File

Update `main.tf` with the S3 bucket, ACL, and data sync resources:

```bash
cat > main.tf << 'EOF'
# New S3 bucket
resource "aws_s3_bucket" "kke_new_bucket" {
  bucket = var.KKE_BUCKET
}

# Make it private
resource "aws_s3_bucket_acl" "kke_new_bucket_acl" {
  bucket = aws_s3_bucket.kke_new_bucket.id
  acl    = "private"
}

# Sync all objects from old bucket to new bucket
resource "null_resource" "s3_sync" {

  provisioner "local-exec" {
    command = "aws --endpoint-url=http://aws:4566 s3 sync s3://devops-s3-4515 s3://${var.KKE_BUCKET}"
  }

  depends_on = [
    aws_s3_bucket.kke_new_bucket,
    aws_s3_bucket_acl.kke_new_bucket_acl
  ]
}
EOF
```

**Purpose:** Provision the new private S3 bucket and orchestrate data migration from the source bucket using an AWS CLI sync command.

**Configuration Breakdown:**

**Resource 1 — S3 Bucket (`aws_s3_bucket.kke_new_bucket`):**
- `bucket = var.KKE_BUCKET` — Names the bucket using the variable (`devops-sync-2251`)

**Resource 2 — S3 Bucket ACL (`aws_s3_bucket_acl.kke_new_bucket_acl`):**
- `bucket = aws_s3_bucket.kke_new_bucket.id` — References the new bucket (implicit dependency)
- `acl = "private"` — Ensures bucket is not publicly accessible
- **Note:** In AWS provider v4+, ACL is a separate resource from the bucket for compliance with S3 Object Ownership defaults

**Resource 3 — Data Sync (`null_resource.s3_sync`):**
- `provisioner "local-exec"` — Runs the specified shell command on the machine executing Terraform
- `command = "aws --endpoint-url=http://aws:4566 s3 sync s3://devops-s3-4515 s3://${var.KKE_BUCKET}"` — Syncs all objects from source to destination:
  - `--endpoint-url=http://aws:4566` — Points to the LocalStack endpoint (used in KKE lab environments)
  - `s3 sync` — Copies only new/changed objects (idempotent) from source to destination
  - `s3://devops-s3-4515` — Source bucket (pre-existing)
  - `s3://${var.KKE_BUCKET}` — Destination bucket (interpolated variable)
- `depends_on` — Ensures **both the bucket and its ACL** are fully created before the sync command runs

**`aws s3 sync` Behavior:**
```
aws s3 sync s3://devops-s3-4515 s3://devops-sync-2251
│
├── Compares source and destination objects
├── Copies new objects (present in source, absent in destination)
├── Copies changed objects (different size or ETag)
└── Does NOT delete destination objects not in source (safe default)
```

**File Content (`main.tf`):**
```hcl
# New S3 bucket
resource "aws_s3_bucket" "kke_new_bucket" {
  bucket = var.KKE_BUCKET
}

# Make it private
resource "aws_s3_bucket_acl" "kke_new_bucket_acl" {
  bucket = aws_s3_bucket.kke_new_bucket.id
  acl    = "private"
}

# Sync all objects from old bucket to new bucket
resource "null_resource" "s3_sync" {

  provisioner "local-exec" {
    command = "aws --endpoint-url=http://aws:4566 s3 sync s3://devops-s3-4515 s3://${var.KKE_BUCKET}"
  }

  depends_on = [
    aws_s3_bucket.kke_new_bucket,
    aws_s3_bucket_acl.kke_new_bucket_acl
  ]
}
```

---

### Step 4: Create Outputs Configuration File

Create `outputs.tf` to export the new bucket name and ACL:

```bash
cat > outputs.tf << 'EOF'
output "new_kke_bucket_name" {
  value = aws_s3_bucket.kke_new_bucket.bucket
}

output "new_kke_bucket_acl" {
  value = aws_s3_bucket_acl.kke_new_bucket_acl.acl
}
EOF
```

**Purpose:** Export the bucket name and ACL for verification after apply.

**Configuration Breakdown:**
- `new_kke_bucket_name` → `aws_s3_bucket.kke_new_bucket.bucket` → returns `"devops-sync-2251"`
- `new_kke_bucket_acl` → `aws_s3_bucket_acl.kke_new_bucket_acl.acl` → returns `"private"`

**File Content (`outputs.tf`):**
```hcl
output "new_kke_bucket_name" {
  value = aws_s3_bucket.kke_new_bucket.bucket
}

output "new_kke_bucket_acl" {
  value = aws_s3_bucket_acl.kke_new_bucket_acl.acl
}
```

---

### Step 5: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Expected Output:**
```
total 16
drwxr-xr-x 2 bob bob 4096 Feb 25 11:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 10:55 ..
-rw-r--r-- 1 bob bob  498 Feb 25 11:00 main.tf
-rw-r--r-- 1 bob bob  148 Feb 25 11:00 outputs.tf
-rw-r--r-- 1 bob bob  118 Feb 25 11:00 variables.tf
```

**Success Indicators:**
- ✅ All three files present
- ✅ No extra `.tf` files created

---

### Step 6: Initialize Terraform

```bash
terraform init
```

**Purpose:** Download the AWS provider and `null` provider plugins.

**Expected Output:**
```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Finding latest version of hashicorp/null...
- Installing hashicorp/aws v5.31.0...
- Installing hashicorp/null v3.2.2...
- Installed hashicorp/aws v5.31.0 (signed by HashiCorp)
- Installed hashicorp/null v3.2.2 (signed by HashiCorp)

Terraform has been successfully initialized!
```

> 📝 **Note:** The `null` provider is automatically detected from the `null_resource` in `main.tf` and installed alongside the AWS provider.

**Success Indicators:**
- ✅ Both AWS and null providers installed
- ✅ `.terraform/` and `.terraform.lock.hcl` created

---

### Step 7: Validate Configuration

```bash
terraform validate
```

**Expected Output:**
```
Success! The configuration is valid.
```

**What's Validated:**
- ✅ `aws_s3_bucket` and `aws_s3_bucket_acl` block attributes are correct
- ✅ `null_resource` with `local-exec` provisioner is valid
- ✅ `depends_on` references are resolvable
- ✅ Variable interpolation in `command` string is correct

---

### Step 8: Format Configuration Files

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

### Step 9: Review Execution Plan

```bash
terraform plan
```

**Purpose:** Preview creation of the S3 bucket + ACL + sync operation.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.kke_new_bucket will be created
  + resource "aws_s3_bucket" "kke_new_bucket" {
      + arn                         = (known after apply)
      + bucket                      = "devops-sync-2251"
      + id                          = (known after apply)
      + region                      = (known after apply)
    }

  # aws_s3_bucket_acl.kke_new_bucket_acl will be created
  + resource "aws_s3_bucket_acl" "kke_new_bucket_acl" {
      + acl    = "private"
      + bucket = (known after apply)
      + id     = (known after apply)
    }

  # null_resource.s3_sync will be created
  + resource "null_resource" "s3_sync" {
      + id = (known after apply)
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + new_kke_bucket_acl  = "private"
  + new_kke_bucket_name = "devops-sync-2251"

───────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
```

**Plan Summary:**
- **Resources to Create:** 3 (S3 bucket + ACL + null_resource sync)
- **Outputs:** Both known at plan time

**Success Indicators:**
- ✅ Bucket named `devops-sync-2251`
- ✅ ACL set to `private`
- ✅ `null_resource.s3_sync` planned (sync will run at apply)
- ✅ Both output values shown

---

### Step 10: Apply Configuration

```bash
terraform apply -auto-approve
```

**Purpose:** Create the bucket, set ACL, then execute the S3 data migration sync.

**Expected Output:**
```
Plan: 3 to add, 0 to change, 0 to destroy.

aws_s3_bucket.kke_new_bucket: Creating...
aws_s3_bucket.kke_new_bucket: Creation complete after 2s [id=devops-sync-2251]
aws_s3_bucket_acl.kke_new_bucket_acl: Creating...
aws_s3_bucket_acl.kke_new_bucket_acl: Creation complete after 1s [id=devops-sync-2251,private]
null_resource.s3_sync: Creating...
null_resource.s3_sync: Provisioning with 'local-exec'...
null_resource.s3_sync (local-exec): Executing: ["/bin/sh" "-c" "aws --endpoint-url=http://aws:4566 s3 sync s3://devops-s3-4515 s3://devops-sync-2251"]
null_resource.s3_sync (local-exec): copy: s3://devops-s3-4515/file1.txt to s3://devops-sync-2251/file1.txt
null_resource.s3_sync (local-exec): copy: s3://devops-s3-4515/data/records.csv to s3://devops-sync-2251/data/records.csv
null_resource.s3_sync (local-exec): copy: s3://devops-s3-4515/logs/app.log to s3://devops-sync-2251/logs/app.log
null_resource.s3_sync: Creation complete after 8s [id=1234567890]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

new_kke_bucket_acl  = "private"
new_kke_bucket_name = "devops-sync-2251"
```

**Success Indicators:**
- ✅ Bucket `devops-sync-2251` created
- ✅ ACL set to `private`
- ✅ `local-exec` runs and shows files being copied (`copy: s3://...`)
- ✅ Both outputs displayed after apply
- ✅ `3 added, 0 changed, 0 destroyed`

**Creation Order (enforced by `depends_on`):**
| Order | Resource | Reason |
|-------|----------|--------|
| 1st | `aws_s3_bucket.kke_new_bucket` | No dependencies |
| 2nd | `aws_s3_bucket_acl.kke_new_bucket_acl` | Needs bucket ID |
| 3rd | `null_resource.s3_sync` | Needs bucket + ACL ready |

---

### Step 11: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Expected Output:**
```
aws_s3_bucket.kke_new_bucket: Refreshing state... [id=devops-sync-2251]
aws_s3_bucket_acl.kke_new_bucket_acl: Refreshing state... [id=devops-sync-2251,private]
null_resource.s3_sync: Refreshing state... [id=1234567890]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ All 3 resources refreshed from state
- ✅ `null_resource` does **not** re-run on plan (it only runs during `apply`)

**⚠️ Important:** This is **required before task submission**.

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Expected Output:**
```
aws_s3_bucket.kke_new_bucket
aws_s3_bucket_acl.kke_new_bucket_acl
null_resource.s3_sync
```

---

### Step 2: Inspect New S3 Bucket

```bash
terraform state show aws_s3_bucket.kke_new_bucket
```

**Expected Output:**
```
# aws_s3_bucket.kke_new_bucket:
resource "aws_s3_bucket" "kke_new_bucket" {
    bucket = "devops-sync-2251"
    id     = "devops-sync-2251"
    region = "us-east-1"
}
```

---

### Step 3: Inspect Bucket ACL

```bash
terraform state show aws_s3_bucket_acl.kke_new_bucket_acl
```

**Expected Output:**
```
# aws_s3_bucket_acl.kke_new_bucket_acl:
resource "aws_s3_bucket_acl" "kke_new_bucket_acl" {
    acl    = "private"
    bucket = "devops-sync-2251"
    id     = "devops-sync-2251,private"
}
```

**Verification Points:**
- ✅ `acl = "private"` confirmed
- ✅ `bucket` references the correct new bucket

---

### Step 4: Verify Data Migration Completeness

```bash
# List all objects in the source bucket
aws --endpoint-url=http://aws:4566 s3 ls s3://devops-s3-4515 --recursive

# List all objects in the destination bucket
aws --endpoint-url=http://aws:4566 s3 ls s3://devops-sync-2251 --recursive
```

**Purpose:** Confirm both buckets contain identical data.

**Expected Output (both commands should produce identical listings):**
```
2024-02-25 10:00:00       1024 file1.txt
2024-02-25 10:00:00       4096 data/records.csv
2024-02-25 10:00:00       2048 logs/app.log
```

**Success Indicators:**
- ✅ Same number of objects in both buckets
- ✅ Same file names, paths, and sizes
- ✅ No missing files in the destination

---

### Step 5: View Output Values

```bash
terraform output
```

**Expected Output:**
```
new_kke_bucket_acl  = "private"
new_kke_bucket_name = "devops-sync-2251"
```

---

### Step 6: Run Sync Verification (Optional)

```bash
# A dry-run sync will show "no objects to copy" if migration is complete
aws --endpoint-url=http://aws:4566 s3 sync s3://devops-s3-4515 s3://devops-sync-2251 --dryrun
```

**Expected Output (when fully synced):**
```
(dryrun) copy: s3://devops-s3-4515/... to s3://devops-sync-2251/...
# OR:
# (no output — all files already exist in destination)
```

If there is no output, all objects are already in sync.

---

## 🔍 Code Analysis

### Resource Dependency Chain

```
aws_s3_bucket.kke_new_bucket
    │  id → bucket ID
    ▼
aws_s3_bucket_acl.kke_new_bucket_acl   (explicit depends_on + implicit via bucket id)
    │  acl = "private"
    ▼
null_resource.s3_sync                   (explicit depends_on on both above)
    │  local-exec: aws s3 sync ...
    ▼
Data migrated: devops-s3-4515 → devops-sync-2251
```

### `null_resource` + `local-exec` Pattern

```hcl
resource "null_resource" "s3_sync" {

  provisioner "local-exec" {
    command = "aws ... s3 sync s3://source s3://destination"
  }

  depends_on = [
    aws_s3_bucket.kke_new_bucket,
    aws_s3_bucket_acl.kke_new_bucket_acl
  ]
}
```

**Key behaviors of `null_resource`:**
| Behavior | Explanation |
|----------|-------------|
| **Runs once** | `local-exec` runs only on `terraform apply`, never on `terraform plan` |
| **State tracking** | Terraform assigns an ID to the `null_resource`; subsequent plans show "No changes" |
| **No re-run on plan** | Once created, re-applying without destroying/recreating won't re-run the command |
| **Force re-run** | Use `terraform apply -replace="null_resource.s3_sync"` to re-trigger the sync |

### `aws s3 sync` Behavior

| Scenario | `s3 sync` Action |
|----------|-----------------|
| Object in source, not in destination | ✅ Copy to destination |
| Object unchanged (same ETag/size) | ⏭️ Skip (already synced) |
| Object changed in source | ✅ Update in destination |
| Object in destination, not in source | ❌ Leave as-is (no deletion by default) |

To also delete objects from destination that don't exist in source, use `--delete`:
```bash
aws s3 sync s3://source s3://destination --delete
```

### Variable Flow

```
variables.tf: default = "devops-sync-2251"
      │
      ▼
main.tf: bucket = var.KKE_BUCKET        → aws_s3_bucket name
         command = "... s3://${var.KKE_BUCKET}"  → sync destination
      │
      ▼
outputs.tf: aws_s3_bucket.kke_new_bucket.bucket → "devops-sync-2251"
            aws_s3_bucket_acl.kke_new_bucket_acl.acl → "private"
```

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate configuration
terraform validate

# 2. Preview plan (3 resources)
terraform plan

# 3. Apply (bucket + ACL + sync)
terraform apply -auto-approve

# 4. Verify all resources in state
terraform state list

# 5. Compare object counts between buckets
aws --endpoint-url=http://aws:4566 s3 ls s3://devops-s3-4515 --recursive | wc -l
aws --endpoint-url=http://aws:4566 s3 ls s3://devops-sync-2251 --recursive | wc -l

# 6. View outputs
terraform output

# 7. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| New bucket name | `devops-sync-2251` | `terraform output new_kke_bucket_name` |
| Bucket ACL | `private` | `terraform output new_kke_bucket_acl` |
| Sync executed | Files copied in apply output | `terraform apply` output |
| Object count match | Same count in both buckets | `aws s3 ls ... --recursive \| wc -l` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: AWS CLI Not Found in `local-exec`

**Error:**
```
null_resource.s3_sync (local-exec): /bin/sh: aws: command not found
```

**Cause:** The AWS CLI is not installed on the machine running Terraform.

**Solution:**
```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version
```

---

### Issue 2: LocalStack Endpoint Not Reachable

**Error:**
```
null_resource.s3_sync (local-exec): Could not connect to the endpoint URL: "http://aws:4566/"
```

**Cause:** The LocalStack container is not running, or the `aws` hostname is not resolvable.

**Solution:**
```bash
# Check if LocalStack is running
curl http://aws:4566/_localstack/health

# If using Docker, verify container is up
docker ps | grep localstack

# If the hostname is different, update the endpoint URL in main.tf
command = "aws --endpoint-url=http://localhost:4566 s3 sync ..."
```

---

### Issue 3: Source Bucket Does Not Exist

**Error:**
```
null_resource.s3_sync (local-exec): An error occurred (NoSuchBucket): The specified bucket does not exist
```

**Cause:** The source bucket `devops-s3-4515` was not pre-created in the environment.

**Solution:**
```bash
# Verify source bucket exists
aws --endpoint-url=http://aws:4566 s3 ls | grep devops-s3-4515

# If it doesn't exist, confirm with your team lead
# The source bucket should be pre-existing per the task requirements
```

---

### Issue 4: Bucket ACL Error (Object Ownership)

**Error:**
```
Error: putting S3 Bucket ACL: AccessControlListNotSupported: The bucket does not allow ACLs
```

**Cause:** AWS accounts created after April 2023 have Object Ownership set to `BucketOwnerEnforced` by default, which disables ACLs.

**Solution:** Add an ownership controls resource before setting ACL:
```hcl
resource "aws_s3_bucket_ownership_controls" "kke_new_bucket_ownership" {
  bucket = aws_s3_bucket.kke_new_bucket.id

  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

resource "aws_s3_bucket_acl" "kke_new_bucket_acl" {
  bucket     = aws_s3_bucket.kke_new_bucket.id
  acl        = "private"
  depends_on = [aws_s3_bucket_ownership_controls.kke_new_bucket_ownership]
}
```

---

### Issue 5: `null_resource` Re-Runs Unnecessarily

**Symptom:** Every `terraform apply` re-runs the `aws s3 sync` command.

**Cause:** A `triggers` map with changing values causes `null_resource` to be replaced on each apply.

**Solution:** Do not add volatile triggers. Without any `triggers` map, the `null_resource` runs only once (on first creation):
```hcl
resource "null_resource" "s3_sync" {
  # No triggers = runs once only
  provisioner "local-exec" { ... }
}
```

To force a re-run when needed, use:
```bash
terraform apply -replace="null_resource.s3_sync"
```

---

## 📚 Best Practices

### 1. Use `null_resource` + `local-exec` for Shell Commands in Terraform
When a task requires running CLI commands as part of infrastructure provisioning (data migration, initialization scripts, etc.), `null_resource` with `local-exec` is the correct Terraform pattern. Avoid embedding shell commands directly in resource arguments.

### 2. Always Set `depends_on` on `null_resource`
The `null_resource` has no knowledge of other resources by default. Explicit `depends_on` ensures all required infrastructure (bucket + ACL) is fully provisioned before the sync command executes.

### 3. Use `aws s3 sync` for Idempotent Data Migration
Unlike `aws s3 cp`, `sync` only transfers new or changed objects. Running it multiple times is safe — it won't duplicate data.

### 4. Verify Object Counts After Migration
Always validate data completeness using object count comparison:
```bash
aws s3 ls s3://source --recursive | wc -l
aws s3 ls s3://destination --recursive | wc -l
```

### 5. Keep Bucket ACL Separate from Bucket Resource (Provider v4+)
Since AWS provider v4+, the bucket ACL must be a separate `aws_s3_bucket_acl` resource — not an inline `acl` argument in `aws_s3_bucket`. This aligns with AWS's push toward Object Ownership controls.

### 6. Force Re-sync Using `-replace` When Needed
If objects change in the source bucket after the initial migration:
```bash
terraform apply -replace="null_resource.s3_sync"
```
This destroys and recreates the `null_resource`, re-running the `local-exec` sync command.

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Name | Configuration |
|----------|------|---------------|
| S3 Bucket | `devops-sync-2251` | Private, variable-driven |
| S3 Bucket ACL | `devops-sync-2251` | `private` |
| Data Sync | `null_resource.s3_sync` | `aws s3 sync devops-s3-4515 → devops-sync-2251` |

### Files Created / Updated

| File | Purpose |
|------|---------|
| `main.tf` | S3 bucket + ACL + `null_resource` sync with `local-exec` |
| `variables.tf` | `KKE_BUCKET` variable with default `devops-sync-2251` |
| `outputs.tf` | Exports `new_kke_bucket_name` and `new_kke_bucket_acl` |

### Task Completion Checklist

- [x] ✅ New private S3 bucket `devops-sync-2251` created
- [x] ✅ Bucket ACL set to `private` via `aws_s3_bucket_acl`
- [x] ✅ All data synced from `devops-s3-4515` to `devops-sync-2251` via `null_resource` + `local-exec`
- [x] ✅ Both buckets contain the same data after migration
- [x] ✅ All resources defined in **single `main.tf`** file
- [x] ✅ `variables.tf` defines `KKE_BUCKET` with default `"devops-sync-2251"`
- [x] ✅ `outputs.tf` exports `new_kke_bucket_name` and `new_kke_bucket_acl`
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply -auto-approve` — **3 resources added, data synced**
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task demonstrates the `null_resource` + `local-exec` pattern — the standard Terraform approach for embedding shell commands into infrastructure pipelines. By chaining `depends_on` from the sync operation to both the bucket and its ACL resource, Terraform guarantees the destination is fully ready before migration begins. The `aws s3 sync` command provides **idempotent, incremental data migration** — safe to re-run and efficient for large datasets. This pattern is widely used for data initialization, configuration bootstrapping, and post-provisioning automation in real-world Terraform deployments.
