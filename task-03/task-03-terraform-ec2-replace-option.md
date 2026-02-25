# 🌟 Task 03 - Forcefully Recreate an EC2 Instance Using Terraform `-replace` Option

## 📌 Task Description

To test **resilience and recreation behavior** in Terraform, the DevOps team needs to demonstrate the use of the `-replace` option to **forcefully destroy and recreate** an EC2 instance without changing its underlying configuration. This simulates real-world scenarios such as instance corruption, OS-level failures, or the need to force a fresh instance boot while keeping the same Terraform configuration.

**Requirements:**
- Use the Terraform CLI **`-replace`** option to destroy and recreate the EC2 instance **`xfusion-ec2`**
- The configuration must remain **unchanged** — only the physical instance is recreated
- The **new instance must have a different Instance ID** than the previously provisioned instance
- Ensure the instance is recreated successfully
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."** after recreation

👉 **Your task:** Demonstrate Terraform's `-replace` lifecycle option by forcefully replacing a running EC2 instance while preserving the original configuration.

💡 **Note:** The `-replace` flag is the modern, safe replacement for the older `terraform taint` workflow. It performs a targeted destroy-and-recreate within the same `apply` operation, ensuring no leftover tainted state if the operation is interrupted.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- AWS EC2 Instance (`xfusion-ec2`) targeted for forced recreation

**Working Directory:** `/home/bob/terraform`

**Key Concept — `-replace` vs Normal Apply:**
| Scenario | Terraform Behavior |
|----------|-------------------|
| Configuration unchanged | `No changes` — nothing happens |
| `terraform apply -replace="resource"` | Force destroy + recreate that specific resource |
| `terraform taint` (deprecated) | Marks resource for replacement on next apply |

---

## 📋 Solution Overview

### 🏗️ What `-replace` Does

When you run `terraform apply -replace="aws_instance.web_server"`, Terraform:

1. **Marks** the specified resource for replacement in the execution plan
2. **Destroys** the existing instance (old Instance ID is released)
3. **Recreates** the instance with the **same configuration** (new Instance ID assigned)
4. **Updates** the state file with the new resource details

This is equivalent to `terraform taint` followed by `terraform apply`, but performed safely in a single atomic operation.

### 🎯 Implementation Strategy
1. Navigate to the Terraform working directory
2. Initialize Terraform (`terraform init`)
3. Apply configuration to create the initial EC2 instance (`terraform apply`)
4. **Note the original Instance ID** for comparison
5. Use `-replace` to forcefully recreate the instance
6. **Verify a new Instance ID** was assigned
7. Run final `terraform plan` to confirm no configuration drift

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

### Step 2: Initialize Terraform

```bash
terraform init
```

**Purpose:** Initialize the working directory, download the AWS provider, and prepare the backend for operations.

**Expected Output:**
```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.31.0...
- Installed hashicorp/aws v5.31.0 (signed by HashiCorp)

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.
```

**Success Indicators:**
- ✅ AWS provider downloaded and installed
- ✅ `.terraform/` directory created
- ✅ `.terraform.lock.hcl` lock file generated

---

### Step 3: Initial Apply — Create the EC2 Instance

```bash
terraform apply -auto-approve
```

**Purpose:** Provision the EC2 instance (`xfusion-ec2`) for the first time, establishing its original Instance ID.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.web_server will be created
  + resource "aws_instance" "web_server" {
      + ami           = "ami-0c02fb55956c7d316"
      + id            = (known after apply)
      + instance_type = "t2.micro"
      + tags          = {
          + "Name" = "xfusion-ec2"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

aws_instance.web_server: Creating...
aws_instance.web_server: Still creating... [10s elapsed]
aws_instance.web_server: Still creating... [20s elapsed]
aws_instance.web_server: Creation complete after 32s [id=i-0abc123def456789a]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

**Success Indicators:**
- ✅ EC2 instance created with an assigned Instance ID
- ✅ Instance Name tag is `xfusion-ec2`
- ✅ Apply completes with `1 added, 0 changed, 0 destroyed`

---

### Step 4: Note the Original Instance ID

After the initial apply, record the original Instance ID. You can retrieve it via:

```bash
terraform state show aws_instance.web_server | grep "^    id"
```

**Expected Output:**
```
    id = "i-0abc123def456789a"
```

**Or using Terraform output (if configured):**
```bash
terraform output
```

> 📝 **Record this Instance ID.** After `-replace`, the new instance **must have a different ID**, confirming it was destroyed and recreated.

---

### Step 5: Forcefully Recreate the EC2 Instance Using `-replace`

```bash
terraform apply -replace="aws_instance.web_server" -auto-approve
```

**Purpose:** Force Terraform to destroy the existing EC2 instance and create a new one with an identical configuration — but a brand-new Instance ID.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # aws_instance.web_server is tainted, so must be replaced
-/+ resource "aws_instance" "web_server" {
      ~ id            = "i-0abc123def456789a" -> (known after apply)  # forces replacement
        ami           = "ami-0c02fb55956c7d316"
        instance_type = "t2.micro"
        tags          = {
            "Name" = "xfusion-ec2"
        }
    }

Plan: 1 to add, 1 to destroy, 0 to change.

aws_instance.web_server: Destroying... [id=i-0abc123def456789a]
aws_instance.web_server: Still destroying... [id=i-0abc123def456789a, 10s elapsed]
aws_instance.web_server: Still destroying... [id=i-0abc123def456789a, 20s elapsed]
aws_instance.web_server: Destruction complete after 30s
aws_instance.web_server: Creating...
aws_instance.web_server: Still creating... [10s elapsed]
aws_instance.web_server: Still creating... [20s elapsed]
aws_instance.web_server: Creation complete after 32s [id=i-0xyz987uvw654321b]

Apply complete! Resources: 1 added, 1 destroyed, 0 changed.
```

**Success Indicators:**
- ✅ Plan symbol shows `-/+` (destroy and then create replacement)
- ✅ Old instance (`i-0abc123def456789a`) is destroyed
- ✅ New instance (`i-0xyz987uvw654321b`) is created
- ✅ Configuration (AMI, type, tags) remains **identical**
- ✅ New Instance ID is **different** from the original

**Understanding the `-/+` Symbol:**
```
+ create          → New resource, no existing one
- destroy         → Resource deleted, not recreated
-/+ replace       → Destroy existing, create new (what -replace does)
~ update in-place → Modify existing resource without recreation
```

---

### Step 6: Verify the New Instance ID

```bash
terraform state show aws_instance.web_server | grep "^    id"
```

**Expected Output:**
```
    id = "i-0xyz987uvw654321b"
```

**Comparison:**
| | Instance ID |
|--|-------------|
| **Before `-replace`** | `i-0abc123def456789a` |
| **After `-replace`** | `i-0xyz987uvw654321b` ✅ Different |

> ✅ The different Instance IDs confirm that the instance was truly destroyed and recreated — not just modified in-place.

---

### Step 7: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Purpose:** Confirm the recreated infrastructure fully matches the Terraform configuration with zero pending changes.

**Expected Output:**
```
aws_instance.web_server: Refreshing state... [id=i-0xyz987uvw654321b]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ New instance ID is reflected in the refreshed state
- ✅ No configuration drift detected
- ✅ Task is ready for submission

**⚠️ Important:** This verification step is **required before task submission**. The "No changes" message confirms the recreated instance fully matches the declared Terraform configuration.

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Purpose:** Confirm the EC2 instance is present in the Terraform state after replacement.

**Expected Output:**
```
aws_instance.web_server
```

**Success Indicators:**
- ✅ `aws_instance.web_server` present in state
- ✅ Only one instance resource (no duplicates)

---

### Step 2: Inspect Full Instance Details

```bash
terraform state show aws_instance.web_server
```

**Purpose:** Confirm all instance attributes are correct and Instance ID is new.

**Expected Output:**
```
# aws_instance.web_server:
resource "aws_instance" "web_server" {
    ami                    = "ami-0c02fb55956c7d316"
    id                     = "i-0xyz987uvw654321b"
    instance_type          = "t2.micro"
    private_ip             = "10.x.x.x"
    tags                   = {
        "Name" = "xfusion-ec2"
    }
}
```

**Verification Points:**
- ✅ Instance ID is the **new** ID (different from original)
- ✅ `ami` and `instance_type` remain unchanged
- ✅ Name tag is `xfusion-ec2`

---

### Step 3: AWS CLI Verification (Optional)

```bash
# Verify new instance is in running state
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].{ID:InstanceId,State:State.Name,Type:InstanceType}" \
  --output table
```

**Expected Output:**
```
----------------------------------------------------
|              DescribeInstances                   |
+----------------------+-----------+--------------+
|          ID          |   State   |    Type      |
+----------------------+-----------+--------------+
|  i-0xyz987uvw654321b |  running  |  t2.micro   |
+----------------------+-----------+--------------+
```

**Purpose:** Independent confirmation via AWS CLI that the new instance is running.

---

## 🔍 Code Analysis

### The `-replace` Option Explained

The `-replace` option was introduced in **Terraform v0.15.2** as a safe, explicit way to force resource recreation.

**Syntax:**
```bash
terraform apply -replace="<resource_type>.<resource_name>"
```

**Example:**
```bash
terraform apply -replace="aws_instance.web_server" -auto-approve
```

### `-replace` vs `terraform taint` (Deprecated)

| Feature | `terraform apply -replace` | `terraform taint` (Deprecated) |
|---------|---------------------------|-------------------------------|
| **Introduced** | Terraform v0.15.2 | Legacy |
| **Status** | ✅ Current best practice | ⚠️ Deprecated in v0.15.2 |
| **How it works** | Single apply operation | Two-step: taint → apply |
| **State safety** | Atomic — no tainted state left if interrupted | Leaves tainted state on interruption |
| **Visibility** | Plan shows `-/+` before applying | Taint is recorded before apply |
| **Recommended** | ✅ Yes | ❌ No |

**Old workflow (deprecated):**
```bash
# Deprecated approach
terraform taint aws_instance.web_server
terraform apply -auto-approve
```

**New workflow (current best practice):**
```bash
# Modern approach
terraform apply -replace="aws_instance.web_server" -auto-approve
```

### What Happens Internally

```
terraform apply -replace="aws_instance.web_server"
        │
        ▼
1. Terraform generates execution plan
        │  Marks resource with -/+ (replace)
        ▼
2. Destroys existing instance
        │  Releases old Instance ID (i-0abc...)
        ▼
3. Creates new instance (same config)
        │  AWS assigns new Instance ID (i-0xyz...)
        ▼
4. Updates terraform.tfstate
        │  Stores new Instance ID
        ▼
5. Final state matches configuration
        │  terraform plan → "No changes"
        ▼
       ✅ Done
```

### Lifecycle of the Instance ID

```
Initial Apply                  After -replace
─────────────                  ──────────────
Instance: xfusion-ec2          Instance: xfusion-ec2
ID: i-0abc123def456789a    →   ID: i-0xyz987uvw654321b  (new)
AMI: ami-0c02fb5...            AMI: ami-0c02fb5...       (same)
Type: t2.micro                 Type: t2.micro            (same)
Tags: Name=xfusion-ec2         Tags: Name=xfusion-ec2    (same)
```

The configuration does not change — only the **physical AWS resource** is destroyed and recreated.

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Record original instance ID
terraform state show aws_instance.web_server | grep "^    id"

# 2. Apply the replace operation
terraform apply -replace="aws_instance.web_server" -auto-approve

# 3. Confirm new instance ID is different
terraform state show aws_instance.web_server | grep "^    id"

# 4. Verify state integrity
terraform state list

# 5. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| Resource in state | `aws_instance.web_server` | `terraform state list` |
| New Instance ID | Different from original | `terraform state show aws_instance.web_server` |
| Instance name tag | `xfusion-ec2` | `terraform state show aws_instance.web_server` |
| Final plan result | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: Resource Not Found for `-replace`

**Error:**
```
Error: No such resource instance

  A managed resource "aws_instance" "web_server" has not been declared...
```

**Cause:** The resource address passed to `-replace` doesn't match the resource name in `main.tf`.

**Solution:**
```bash
# List all resources in state to find the correct resource address
terraform state list

# Use the exact address shown in the output
terraform apply -replace="aws_instance.web_server" -auto-approve
```

---

### Issue 2: Instance Takes Too Long to Destroy

**Symptom:**
```
aws_instance.web_server: Still destroying... [id=i-0abc..., 5m0s elapsed]
```

**Cause:** EC2 instance may have termination protection enabled or be part of an Auto Scaling Group.

**Solution:**
```bash
# Check termination protection via AWS CLI
aws ec2 describe-instance-attribute \
  --instance-id i-0abc123def456789a \
  --attribute disableApiTermination

# If enabled, disable it
aws ec2 modify-instance-attribute \
  --instance-id i-0abc123def456789a \
  --no-disable-api-termination
```

---

### Issue 3: Same Instance ID After `-replace`

**Symptom:** The Instance ID didn't change after running `-replace`.

**Cause:** The `-replace` operation may have been interrupted or the plan was not executed correctly.

**Solution:**
```bash
# Confirm the plan shows -/+ before applying
terraform plan -replace="aws_instance.web_server"

# Ensure you see the -/+ destroy-and-create symbol
# Then apply
terraform apply -replace="aws_instance.web_server" -auto-approve
```

---

### Issue 4: Terraform State Drift After Interruption

**Error:**
```
Error: Instance has been replaced but state is inconsistent
```

**Cause:** If the `-replace` apply was interrupted mid-way, the state file may be inconsistent.

**Solution:**
```bash
# Refresh state to sync with actual AWS resources
terraform refresh

# If the old instance no longer exists, remove it from state
terraform state rm aws_instance.web_server

# Then apply to recreate from scratch
terraform apply -auto-approve
```

---

### Issue 5: Plan Shows Changes After `-replace`

**Error:**
```
Plan: 0 to add, 1 to change, 0 to destroy.
```

**Cause:** There may be configuration drift — an attribute in the new instance differs from what Terraform expects.

**Solution:**
```bash
# Review what changed
terraform plan

# If intentional drift, update main.tf to match
# If unintentional, re-apply to correct
terraform apply -auto-approve
```

---

## 📚 Best Practices

### 1. Always Use `-replace` Instead of `terraform taint`
`terraform taint` is **deprecated since v0.15.2**. Always use the `-replace` flag for forced resource recreation — it's atomic, safer, and more explicit.

### 2. Note the Instance ID Before Replacing
Document the original Instance ID before running `-replace`. This provides proof that a genuine replacement occurred — critical for auditing and compliance.

### 3. Preview the Plan Before Applying
```bash
# Preview what -replace will do without applying
terraform plan -replace="aws_instance.web_server"
```
Reviewing the plan first ensures you're replacing the correct resource and confirms the `-/+` replace action will be executed.

### 4. Use `-replace` for Specific Resources Only
Avoid running `terraform apply -replace` with broad patterns. Always specify the exact resource address to prevent unintended replacements in large infrastructures.

### 5. Verify with `terraform plan` After Replacement
Always run `terraform plan` as the final step after any `-replace` operation. The "No changes" result is your confirmation that:
- The new instance is healthy and registered in AWS
- The Terraform state matches the actual infrastructure
- No additional changes are needed

### 6. Use for These Common Real-World Scenarios
- **OS-level corruption:** Instance is running but OS is unresponsive
- **Compliance requirements:** Fresh instance required for audit trails
- **AMI rollout testing:** Verify that re-instantiation with the same AMI works correctly
- **Spot instance replacement:** Manual recreation without changing underlying configuration

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Phase | Action | Result |
|-------|--------|--------|
| **Initial Provision** | `terraform apply -auto-approve` | EC2 `xfusion-ec2` created with Instance ID `i-0abc...` |
| **Force Replace** | `terraform apply -replace="aws_instance.web_server"` | Instance destroyed and recreated with new ID `i-0xyz...` |
| **Verification** | `terraform plan` | "No changes. Your infrastructure matches the configuration." |

### Instance ID Comparison

| | Instance ID | Configuration |
|--|-------------|---------------|
| **Original Instance** | `i-0abc123def456789a` | `t2.micro` / `xfusion-ec2` |
| **Replaced Instance** | `i-0xyz987uvw654321b` ✅ Different | `t2.micro` / `xfusion-ec2` (unchanged) |

### Task Completion Checklist

- [x] ✅ Terraform initialized successfully with `terraform init`
- [x] ✅ Initial EC2 instance `xfusion-ec2` provisioned with `terraform apply`
- [x] ✅ Original Instance ID noted before replacement
- [x] ✅ Instance forcefully recreated using `terraform apply -replace="aws_instance.web_server"`
- [x] ✅ Plan clearly showed `-/+` (destroy and recreate) symbol
- [x] ✅ New instance has a **different Instance ID** than the original
- [x] ✅ EC2 instance name tag remains `xfusion-ec2` (configuration unchanged)
- [x] ✅ `terraform state list` shows `aws_instance.web_server` in state
- [x] ✅ Final `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** The `terraform apply -replace` flag is a powerful operational tool for **forceful resource recreation without configuration changes**. It's the modern, safe successor to the deprecated `terraform taint` workflow and is essential knowledge for any DevOps engineer managing AWS infrastructure with Terraform. The distinctive `-/+` plan symbol and the resulting new Instance ID are the definitive proof that a true replacement occurred.
