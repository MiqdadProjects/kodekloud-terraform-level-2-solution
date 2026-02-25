# 🌟 Task 06 - Create an AMI from an EC2 Instance and Launch a New Instance Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** needs to create an **Amazon Machine Image (AMI)** from an existing EC2 instance for **backup and scaling purposes**. This pattern is commonly used in production environments to capture the current state of an instance (including installed software, configurations, and data) as a reusable image, then launch new instances from that image for horizontal scaling or environment duplication.

**Requirements:**
- An existing EC2 instance named **`devops-ec2`** is already present
- Create an **AMI** named **`devops-ec2-ami`** from the existing `devops-ec2` instance
- Launch a **new EC2 instance** named **`devops-ec2-new`** using that AMI
- Update the **`main.tf`** file (do not create a separate `.tf` file) to provision both the AMI and the new instance
- Use an **`outputs.tf`** file with:
  - `KKE_ami_id` — the ID of the created AMI
  - `KKE_new_instance_id` — the Instance ID of the new EC2 instance
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Capture the existing EC2 instance as an AMI using `aws_ami_from_instance` and launch a new EC2 from that AMI — implementing a full instance cloning and re-launch pipeline in Terraform.

💡 **Note:** The `aws_ami_from_instance` resource creates an AMI by snapshotting the root volume (and any other attached volumes) of the source instance. The source instance must be **running or stopped** — not terminated. Explicit `depends_on` is used to enforce strict creation ordering across these three chained resources.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- Existing EC2 Instance (`devops-ec2`) — source for AMI creation
- AMI (`devops-ec2-ami`) — image captured from the source instance
- New EC2 Instance (`devops-ec2-new`) — launched from the created AMI

**Working Directory:** `/home/bob/terraform`

**Resource Pipeline:**
```
devops-ec2 (existing instance)
      │
      ▼  aws_ami_from_instance
devops-ec2-ami (AMI snapshot)
      │
      ▼  aws_instance
devops-ec2-new (new instance from AMI)
```

**Resource Summary:**
| Resource | Terraform Type | Name | Details |
|----------|---------------|------|---------|
| Source Instance | `aws_instance` | `devops-ec2` | `t2.micro`, existing |
| AMI | `aws_ami_from_instance` | `devops-ec2-ami` | Captured from source instance |
| New Instance | `aws_instance` | `devops-ec2-new` | `t2.micro`, launched from AMI |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_instance` (source):** Defines the existing `devops-ec2` instance in Terraform state
- **`aws_ami_from_instance`:** Creates an AMI by snapshotting the source instance — requires instance ID and creates explicit dependency
- **`aws_instance` (new):** Launches `devops-ec2-new` using the AMI ID from `aws_ami_from_instance`
- **Explicit `depends_on`:** Used at each stage to guarantee strict provisioning order across the three chained resources

### 🎯 Implementation Strategy
1. Update `main.tf` with all three resources and explicit dependencies
2. Create `outputs.tf` to export AMI ID and new instance ID
3. Initialize, validate, format, plan, apply, and verify

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf      # Source EC2 + AMI creation + new EC2 from AMI
└── outputs.tf   # Output: AMI ID and new instance ID
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

**Purpose:** Ensure all Terraform commands execute in the correct working directory.

**Verification:**
```bash
pwd
```

**Expected Output:**
```
/home/bob/terraform
```

---

### Step 2: Update Main Configuration File

Update `main.tf` with all three resources — source instance, AMI, and new instance:

```bash
cat > main.tf << 'EOF'
# Existing EC2 instance
resource "aws_instance" "ec2" {
  ami           = "ami-0c101f26f147fa7fd"
  instance_type = "t2.micro"

  vpc_security_group_ids = [
    "sg-e5591365717e1f96e"
  ]

  tags = {
    Name = "devops-ec2"
  }
}

# Create AMI from existing EC2 instance
resource "aws_ami_from_instance" "devops_ami" {
  name               = "devops-ec2-ami"
  source_instance_id = aws_instance.ec2.id

  depends_on = [aws_instance.ec2]
}

# Launch new EC2 instance from the AMI
resource "aws_instance" "devops_ec2_new" {
  ami           = aws_ami_from_instance.devops_ami.id
  instance_type = "t2.micro"

  vpc_security_group_ids = [
    "sg-e5591365717e1f96e"
  ]

  tags = {
    Name = "devops-ec2-new"
  }

  depends_on = [aws_ami_from_instance.devops_ami]
}
EOF
```

**Purpose:** Define the complete three-resource pipeline — source instance, AMI creation, and new instance launch — in a single `main.tf` file.

**Configuration Breakdown:**

**Resource 1 — Source EC2 Instance (`aws_instance.ec2`):**
- `ami = "ami-0c101f26f147fa7fd"` — The base AMI used to launch the original `devops-ec2` instance
- `instance_type = "t2.micro"` — Instance size for the source instance
- `vpc_security_group_ids` — Attaches the existing security group (pre-existing in AWS)
- `tags.Name = "devops-ec2"` — Matches the existing instance name so Terraform manages it

**Resource 2 — AMI from Instance (`aws_ami_from_instance.devops_ami`):**
- `name = "devops-ec2-ami"` — Name of the AMI to be created in AWS
- `source_instance_id = aws_instance.ec2.id` — References the source instance ID (also creates an implicit dependency)
- `depends_on = [aws_instance.ec2]` — **Explicit dependency** ensuring the EC2 instance is fully available before AMI creation begins
- **Key behavior:** AWS creates EBS snapshots of all volumes attached to the source instance and bundles them into the AMI

**Resource 3 — New EC2 from AMI (`aws_instance.devops_ec2_new`):**
- `ami = aws_ami_from_instance.devops_ami.id` — Uses the newly created AMI's ID (implicit dependency)
- `instance_type = "t2.micro"` — Same instance type as source
- `vpc_security_group_ids` — Same security group for consistent network access
- `depends_on = [aws_ami_from_instance.devops_ami]` — **Explicit dependency** ensuring AMI is fully registered before launching new instance
- `tags.Name = "devops-ec2-new"` — Unique name for the new instance

**Dependency Chain:**
| Stage | Resource | Explicit `depends_on` |
|-------|----------|----------------------|
| 1st | `aws_instance.ec2` | None |
| 2nd | `aws_ami_from_instance.devops_ami` | `[aws_instance.ec2]` |
| 3rd | `aws_instance.devops_ec2_new` | `[aws_ami_from_instance.devops_ami]` |

**File Content (`main.tf`):**
```hcl
# Existing EC2 instance
resource "aws_instance" "ec2" {
  ami           = "ami-0c101f26f147fa7fd"
  instance_type = "t2.micro"

  vpc_security_group_ids = [
    "sg-e5591365717e1f96e"
  ]

  tags = {
    Name = "devops-ec2"
  }
}

# Create AMI from existing EC2 instance
resource "aws_ami_from_instance" "devops_ami" {
  name               = "devops-ec2-ami"
  source_instance_id = aws_instance.ec2.id

  depends_on = [aws_instance.ec2]
}

# Launch new EC2 instance from the AMI
resource "aws_instance" "devops_ec2_new" {
  ami           = aws_ami_from_instance.devops_ami.id
  instance_type = "t2.micro"

  vpc_security_group_ids = [
    "sg-e5591365717e1f96e"
  ]

  tags = {
    Name = "devops-ec2-new"
  }

  depends_on = [aws_ami_from_instance.devops_ami]
}
```

---

### Step 3: Create Outputs Configuration File

Create `outputs.tf` to export the AMI ID and new instance ID:

```bash
cat > outputs.tf << 'EOF'
output "KKE_ami_id" {
  description = "AMI ID created from devops-ec2"
  value       = aws_ami_from_instance.devops_ami.id
}

output "KKE_new_instance_id" {
  description = "Instance ID of the new EC2 created from AMI"
  value       = aws_instance.devops_ec2_new.id
}
EOF
```

**Purpose:** Export key resource identifiers for verification and downstream use.

**Configuration Breakdown:**
- `KKE_ami_id` — Exports the AMI ID (`ami-xxxx...`) from `aws_ami_from_instance.devops_ami.id`
- `KKE_new_instance_id` — Exports the new EC2 instance ID (`i-xxxx...`) from `aws_instance.devops_ec2_new.id`

**File Content (`outputs.tf`):**
```hcl
output "KKE_ami_id" {
  description = "AMI ID created from devops-ec2"
  value       = aws_ami_from_instance.devops_ami.id
}

output "KKE_new_instance_id" {
  description = "Instance ID of the new EC2 created from AMI"
  value       = aws_instance.devops_ec2_new.id
}
```

---

### Step 4: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Expected Output:**
```
total 12
drwxr-xr-x 2 bob bob 4096 Feb 25 11:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 10:55 ..
-rw-r--r-- 1 bob bob  645 Feb 25 11:00 main.tf
-rw-r--r-- 1 bob bob  248 Feb 25 11:00 outputs.tf
```

**Success Indicators:**
- ✅ Both files present: `main.tf` and `outputs.tf`
- ✅ No extra `.tf` files created

---

### Step 5: Initialize Terraform

```bash
terraform init
```

**Purpose:** Initialize the working directory and download the AWS provider.

**Expected Output:**
```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.31.0...
- Installed hashicorp/aws v5.31.0 (signed by HashiCorp)

Terraform has been successfully initialized!
```

**Success Indicators:**
- ✅ AWS provider downloaded
- ✅ `.terraform/` directory and `.terraform.lock.hcl` created

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
- ✅ `aws_ami_from_instance` block attributes are correct
- ✅ `source_instance_id` correctly references `aws_instance.ec2.id`
- ✅ New instance `ami` correctly references `aws_ami_from_instance.devops_ami.id`
- ✅ `depends_on` syntax is valid

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

**Purpose:** Preview the three-stage creation plan.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_ami_from_instance.devops_ami will be created
  + resource "aws_ami_from_instance" "devops_ami" {
      + id                  = (known after apply)
      + manage_ebs_snapshots = true
      + name                = "devops-ec2-ami"
      + public              = (known after apply)
      + source_instance_id  = (known after apply)
    }

  # aws_instance.devops_ec2_new will be created
  + resource "aws_instance" "devops_ec2_new" {
      + ami           = (known after apply)
      + id            = (known after apply)
      + instance_type = "t2.micro"
      + tags          = {
          + "Name" = "devops-ec2-new"
        }
    }

  # aws_instance.ec2 will be created
  + resource "aws_instance" "ec2" {
      + ami           = "ami-0c101f26f147fa7fd"
      + id            = (known after apply)
      + instance_type = "t2.micro"
      + tags          = {
          + "Name" = "devops-ec2"
        }
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + KKE_ami_id          = (known after apply)
  + KKE_new_instance_id = (known after apply)
```

**Plan Summary:**
- **Resources to Create:** 3 (source EC2 + AMI + new EC2)
- **Outputs:** `KKE_ami_id` and `KKE_new_instance_id` (both known after apply)

**Success Indicators:**
- ✅ All 3 resources planned for creation
- ✅ Correct `depends_on` chain implied in creation order
- ✅ Both outputs shown

---

### Step 9: Apply Configuration

```bash
terraform apply -auto-approve
```

**Purpose:** Execute the pipeline — provision source instance, create AMI, launch new instance.

**Expected Output:**
```
Plan: 3 to add, 0 to change, 0 to destroy.

aws_instance.ec2: Creating...
aws_instance.ec2: Still creating... [10s elapsed]
aws_instance.ec2: Creation complete after 32s [id=i-0source11111111111]

aws_ami_from_instance.devops_ami: Creating...
aws_ami_from_instance.devops_ami: Still creating... [10s elapsed]
aws_ami_from_instance.devops_ami: Still creating... [1m0s elapsed]
aws_ami_from_instance.devops_ami: Creation complete after 2m14s [id=ami-0devops22222222222]

aws_instance.devops_ec2_new: Creating...
aws_instance.devops_ec2_new: Still creating... [10s elapsed]
aws_instance.devops_ec2_new: Creation complete after 33s [id=i-0new33333333333333]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

KKE_ami_id          = "ami-0devops22222222222"
KKE_new_instance_id = "i-0new33333333333333"
```

> ⏱️ **Note:** AMI creation (`aws_ami_from_instance`) is the slowest step — it typically takes **1–5 minutes** as AWS must snapshot all attached EBS volumes. This is expected behavior.

**Success Indicators:**
- ✅ Source EC2 created first (Stage 1)
- ✅ AMI created after source instance is ready (Stage 2)
- ✅ New EC2 launched after AMI is fully registered (Stage 3)
- ✅ Both output values populated with real AWS IDs
- ✅ `3 added, 0 changed, 0 destroyed`

**Creation Order Verification:**
| Stage | Resource | Wait Time |
|-------|----------|-----------|
| 1st | `aws_instance.ec2` (`devops-ec2`) | ~30 seconds |
| 2nd | `aws_ami_from_instance.devops_ami` | 1–5 minutes (EBS snapshot) |
| 3rd | `aws_instance.devops_ec2_new` | ~30 seconds |

---

### Step 10: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Expected Output:**
```
aws_instance.ec2: Refreshing state... [id=i-0source11111111111]
aws_ami_from_instance.devops_ami: Refreshing state... [id=ami-0devops22222222222]
aws_instance.devops_ec2_new: Refreshing state... [id=i-0new33333333333333]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ All 3 resources refreshed from AWS
- ✅ Task is ready for submission

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Expected Output:**
```
aws_ami_from_instance.devops_ami
aws_instance.devops_ec2_new
aws_instance.ec2
```

---

### Step 2: Inspect AMI Details

```bash
terraform state show aws_ami_from_instance.devops_ami
```

**Expected Output:**
```
# aws_ami_from_instance.devops_ami:
resource "aws_ami_from_instance" "devops_ami" {
    id                   = "ami-0devops22222222222"
    manage_ebs_snapshots = true
    name                 = "devops-ec2-ami"
    source_instance_id   = "i-0source11111111111"
}
```

**Verification Points:**
- ✅ `name = "devops-ec2-ami"`
- ✅ `source_instance_id` references the source EC2 instance
- ✅ `id` is a valid AMI ID (`ami-xxxx...`)

---

### Step 3: Inspect New EC2 Instance Details

```bash
terraform state show aws_instance.devops_ec2_new
```

**Expected Output:**
```
# aws_instance.devops_ec2_new:
resource "aws_instance" "devops_ec2_new" {
    ami           = "ami-0devops22222222222"
    id            = "i-0new33333333333333"
    instance_type = "t2.micro"
    tags          = {
        "Name" = "devops-ec2-new"
    }
}
```

**Verification Points:**
- ✅ `ami` matches the AMI created by `aws_ami_from_instance`
- ✅ `tags.Name = "devops-ec2-new"`
- ✅ `instance_type = "t2.micro"`

---

### Step 4: View Output Values

```bash
terraform output
```

**Expected Output:**
```
KKE_ami_id          = "ami-0devops22222222222"
KKE_new_instance_id = "i-0new33333333333333"
```

**Individual Output Access:**
```bash
terraform output KKE_ami_id
terraform output KKE_new_instance_id
```

---

### Step 5: AWS CLI Verification (Optional)

```bash
# Verify the AMI exists and is available
aws ec2 describe-images \
  --image-ids ami-0devops22222222222 \
  --query "Images[].{ID:ImageId,Name:Name,State:State}" \
  --output table

# Verify new EC2 instance is running
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2-new" \
  --query "Reservations[].Instances[].{ID:InstanceId,State:State.Name,AMI:ImageId}" \
  --output table
```

---

## 🔍 Code Analysis

### Three-Stage Resource Pipeline

```
Stage 1: Source Instance
────────────────────────
aws_instance.ec2
  ami           = "ami-0c101f26f147fa7fd"  (base OS image)
  instance_type = "t2.micro"
  tags.Name     = "devops-ec2"

        │  depends_on ensures Stage 1 completes first
        ▼

Stage 2: AMI Creation
─────────────────────
aws_ami_from_instance.devops_ami
  name               = "devops-ec2-ami"
  source_instance_id = aws_instance.ec2.id  → all volumes snapshotted

        │  depends_on ensures Stage 2 completes first
        ▼

Stage 3: New Instance from AMI
──────────────────────────────
aws_instance.devops_ec2_new
  ami           = aws_ami_from_instance.devops_ami.id  (the new AMI)
  instance_type = "t2.micro"
  tags.Name     = "devops-ec2-new"
```

### Explicit vs Implicit Dependencies in This Task

```hcl
# aws_ami_from_instance.devops_ami has BOTH:

# Implicit dependency (via attribute reference):
source_instance_id = aws_instance.ec2.id   ← Terraform detects this automatically

# Explicit dependency (belt-and-suspenders for AMI creation):
depends_on = [aws_instance.ec2]            ← Guarantees full instance readiness
```

Using both is intentional: `source_instance_id` creates an implicit dependency, while `depends_on` explicitly signals that the EC2 instance must be **fully provisioned and stable** before AMI creation begins — not just that its ID is available.

### `aws_ami_from_instance` Resource Explained

| Attribute | Description |
|-----------|-------------|
| `name` | Name of the new AMI in AWS |
| `source_instance_id` | ID of the EC2 instance to snapshot |
| `.id` (output) | The resulting AMI ID (e.g., `ami-xxxx`) |
| `manage_ebs_snapshots` | If `true`, Terraform manages and deletes the EBS snapshots when the AMI is destroyed |

### Output Values

| Output | Source | Type | Example |
|--------|--------|------|---------|
| `KKE_ami_id` | `aws_ami_from_instance.devops_ami.id` | string | `ami-0devops22222222...` |
| `KKE_new_instance_id` | `aws_instance.devops_ec2_new.id` | string | `i-0new333333333333...` |

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate syntax
terraform validate

# 2. Preview the 3-resource plan
terraform plan

# 3. Apply all resources
terraform apply -auto-approve

# 4. List all resources in state
terraform state list

# 5. Confirm AMI details
terraform state show aws_ami_from_instance.devops_ami

# 6. Confirm new instance uses AMI
terraform state show aws_instance.devops_ec2_new

# 7. View outputs
terraform output

# 8. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| Source instance name | `devops-ec2` | `terraform state show aws_instance.ec2` |
| AMI name | `devops-ec2-ami` | `terraform state show aws_ami_from_instance.devops_ami` |
| AMI source instance | Source EC2 ID | `terraform state show aws_ami_from_instance.devops_ami` |
| New instance AMI | Matches AMI ID | `terraform state show aws_instance.devops_ec2_new` |
| New instance name | `devops-ec2-new` | `terraform state show aws_instance.devops_ec2_new` |
| AMI ID output | Valid `ami-xxxx` | `terraform output KKE_ami_id` |
| New instance ID output | Valid `i-xxxx` | `terraform output KKE_new_instance_id` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: AMI Creation Takes Too Long or Times Out

**Symptom:**
```
aws_ami_from_instance.devops_ami: Still creating... [5m0s elapsed]
```

**Cause:** AMI creation involves EBS volume snapshotting, which takes longer for larger volumes or instances with many attached volumes.

**Solution:**
```bash
# AMI creation can take up to 10 minutes — this is normal
# Check AMI status in AWS Console: EC2 → AMIs
aws ec2 describe-images \
  --filters "Name=name,Values=devops-ec2-ami" \
  --query "Images[].{ID:ImageId,State:State}" \
  --output table
```
Wait for State to change from `pending` to `available`.

---

### Issue 2: Existing Instance Not Found

**Error:**
```
Error: no matching EC2 Instance found
```

**Cause:** The `devops-ec2` instance does not exist in AWS, or its AMI/security group ID in `main.tf` doesn't match the actual running instance.

**Solution:**
```bash
# Get actual AMI and SG IDs of the existing instance
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[].Instances[].{AMI:ImageId,SG:SecurityGroups[0].GroupId}" \
  --output table

# Update main.tf with the correct AMI ID and security group ID
```

---

### Issue 3: AMI Name Already Exists

**Error:**
```
Error: InvalidAMIName.Duplicate: AMI name 'devops-ec2-ami' is already in use by AMI ami-0EXISTING
```

**Cause:** An AMI with the same name already exists in the account.

**Solution:**
```bash
# Find and deregister the existing AMI
aws ec2 describe-images --filters "Name=name,Values=devops-ec2-ami" --query "Images[].ImageId" --output text

# Deregister it (replace with actual AMI ID)
aws ec2 deregister-image --image-id ami-0EXISTING

# Then re-apply
terraform apply -auto-approve
```

---

### Issue 4: New Instance Fails to Launch from AMI

**Error:**
```
Error: creating EC2 Instance: InvalidAMIID.Unavailable: The AMI is not available
```

**Cause:** The AMI may still be in `pending` state when Terraform tries to launch the new instance.

**Solution:** This should be prevented by the explicit `depends_on = [aws_ami_from_instance.devops_ami]`. If it still occurs, verify the AMI state:
```bash
aws ec2 describe-images --image-ids <AMI_ID> --query "Images[].State" --output text
# Should return "available" before launching instances
```

---

### Issue 5: Security Group ID Not Found

**Error:**
```
Error: creating EC2 Instance: InvalidGroup.NotFound: The security group 'sg-e5591365717e1f96e' does not exist
```

**Cause:** The hardcoded security group ID in `main.tf` doesn't exist in the target AWS account or region.

**Solution:**
```bash
# List security groups in your account
aws ec2 describe-security-groups \
  --query "SecurityGroups[].{ID:GroupId,Name:GroupName}" \
  --output table

# Update main.tf vpc_security_group_ids with a valid SG ID
```

---

## 📚 Best Practices

### 1. Always Use `depends_on` for AMI-Based Pipelines
Even though `source_instance_id` creates an implicit dependency, the explicit `depends_on` ensures Terraform waits for the **full resource lifecycle** to complete — not just ID assignment — before proceeding to the next stage.

### 2. Stop the Source Instance Before AMI Creation (Production)
For data consistency in production, stop the source instance before creating an AMI:
```bash
aws ec2 stop-instances --instance-ids i-0source...
# Wait for stopped state, then trigger terraform apply
```
Terraform's `aws_ami_from_instance` can create AMIs from running instances, but stopping first avoids filesystem inconsistencies.

### 3. Tag AMIs for Lifecycle Management
```hcl
resource "aws_ami_from_instance" "devops_ami" {
  # ...
  tags = {
    Name        = "devops-ec2-ami"
    CreatedFrom = "devops-ec2"
    Environment = "production"
    CreatedAt   = timestamp()
  }
}
```
Tagged AMIs are easier to track, audit, and clean up over time.

### 4. Output AMI ID for Downstream Reuse
```hcl
output "KKE_ami_id" {
  value = aws_ami_from_instance.devops_ami.id
}
```
The AMI ID output allows other Terraform workspaces, pipelines, or Auto Scaling Groups to reference this AMI without hardcoding.

### 5. Clean Up AMIs When No Longer Needed
AMIs consume EBS snapshot storage, which incurs costs. Always deregister unused AMIs and delete their associated snapshots.

### 6. Use `terraform destroy` Carefully with AMI Resources
When `manage_ebs_snapshots = true` (default), `terraform destroy` will **deregister the AMI and delete its EBS snapshots**. Ensure the AMI is not in use by other instances before destroying.

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Stage | Resource | Name | Result |
|-------|----------|------|--------|
| 1 | `aws_instance.ec2` | `devops-ec2` | Source instance provisioned |
| 2 | `aws_ami_from_instance.devops_ami` | `devops-ec2-ami` | AMI created from source instance |
| 3 | `aws_instance.devops_ec2_new` | `devops-ec2-new` | New instance launched from AMI |

### Files Created / Updated

| File | Purpose |
|------|---------|
| `main.tf` | All 3 resources: source EC2 + AMI + new EC2 with explicit `depends_on` chain |
| `outputs.tf` | Exports `KKE_ami_id` and `KKE_new_instance_id` |

### Task Completion Checklist

- [x] ✅ Source EC2 instance `devops-ec2` defined and provisioned in `main.tf`
- [x] ✅ AMI `devops-ec2-ami` created from `devops-ec2` using `aws_ami_from_instance`
- [x] ✅ New EC2 instance `devops-ec2-new` launched from the created AMI
- [x] ✅ All resources defined in **single `main.tf`** file
- [x] ✅ `depends_on` chain enforces strict 3-stage creation order
- [x] ✅ `outputs.tf` exports `KKE_ami_id` (AMI ID)
- [x] ✅ `outputs.tf` exports `KKE_new_instance_id` (new instance ID)
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply -auto-approve` — **3 resources added**
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task demonstrates a **full instance cloning pipeline** using Terraform — capturing an existing EC2 instance as an AMI with `aws_ami_from_instance` and immediately launching a new instance from it. The explicit `depends_on` chain at each stage is essential because AMI creation and registration takes significant time, and Terraform must wait for each stage to fully complete before proceeding. This pattern is the foundation of **golden image workflows**, **blue-green deployments**, and **horizontal scaling pipelines** in production AWS environments.
