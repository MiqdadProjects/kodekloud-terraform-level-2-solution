# 🌟 Task 05 - Provision an EC2 Instance with an Elastic IP Using Terraform

## 📌 Task Description

The **Nautilus DevOps Team** has received a request from the **Development Team** to provision a new EC2 instance paired with a **static, consistent public IP address**. By default, EC2 instances receive a dynamic public IP that changes every time the instance is stopped and restarted. To ensure the application has a **stable, reliable access point**, an **Elastic IP (EIP)** address must be provisioned and associated with the instance.

**Requirements:**
- Create an EC2 instance named **`datacenter-ec2`** using a **Linux AMI** (Ubuntu)
- Instance type must be **`t2.micro`**
- Provision an **Elastic IP** named **`datacenter-eip`** and associate it with the instance
- All resources must be defined in a single **`main.tf`** file (no additional `.tf` resource files)
- Use an **`outputs.tf`** file with:
  - `KKE_instance_name` — the EC2 instance name tag
  - `KKE_eip` — the allocated Elastic IP address
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Provision a production-ready EC2 instance with a persistent public IP address using Terraform, ensuring consistent accessibility for hosted applications.

💡 **Note:** An Elastic IP address is a **static IPv4 address** designed for dynamic cloud computing. It remains allocated to your AWS account until explicitly released, ensuring the application endpoint does not change even if the instance is stopped and restarted.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- AWS EC2 Instance (`datacenter-ec2`) — Ubuntu, `t2.micro`
- AWS Elastic IP (`datacenter-eip`) — static public IP associated with the instance

**Working Directory:** `/home/bob/terraform`

**Network Configuration:**
| Resource | Name | Details |
|----------|------|---------|
| EC2 Instance | `datacenter-ec2` | `t2.micro`, Ubuntu |
| Elastic IP | `datacenter-eip` | Static public IPv4, auto-associated |

**Why Elastic IP?**
| IP Type | Behavior | Use Case |
|---------|----------|----------|
| **Default Public IP** | Changes on every stop/start | Development, short-lived workloads |
| **Elastic IP** | Static — never changes | Production apps, DNS records, SSL certs |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **EC2 Instance (`aws_instance`):** Ubuntu `t2.micro` compute instance
- **Elastic IP (`aws_eip`):** Static public IP associated with the EC2 instance via `instance` attribute
- **Implicit Dependency:** The EIP references `aws_instance.datacenter_ec2.id`, so Terraform automatically provisions the EC2 instance before allocating and associating the EIP

### 🎯 Implementation Strategy
1. Create `main.tf` with EC2 instance and EIP resources
2. Create `outputs.tf` to export instance name and EIP address
3. Initialize, validate, format, plan, apply, and verify

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf      # EC2 Instance + Elastic IP resource definitions
└── outputs.tf   # Output: instance name and EIP address
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

**Purpose:** Ensure all Terraform commands run in the correct working directory.

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

Create `main.tf` with the EC2 instance and Elastic IP resources:

```bash
cat > main.tf << 'EOF'
# EC2 INSTANCE
resource "aws_instance" "datacenter_ec2" {
  ami           = "ami-0c02fb55956c7d316" # Ubuntu Linux AMI
  instance_type = "t2.micro"

  tags = {
    Name = "datacenter-ec2"
  }
}

# ELASTIC IP
resource "aws_eip" "datacenter_eip" {
  instance = aws_instance.datacenter_ec2.id

  tags = {
    Name = "datacenter-eip"
  }
}
EOF
```

**Purpose:** Define both infrastructure resources — the EC2 instance and its associated Elastic IP — in a single configuration file.

**Configuration Breakdown:**

**EC2 Instance Resource (`aws_instance.datacenter_ec2`):**
- `ami = "ami-0c02fb55956c7d316"` — Ubuntu Linux AMI for us-east-1 region
- `instance_type = "t2.micro"` — Cost-effective general-purpose instance as required
- `tags.Name = "datacenter-ec2"` — Name tag used for identification and output

**Elastic IP Resource (`aws_eip.datacenter_eip`):**
- `instance = aws_instance.datacenter_ec2.id` — Associates the EIP with the EC2 instance by ID; also creates an **implicit dependency** ensuring the EC2 instance is created before the EIP
- `tags.Name = "datacenter-eip"` — Labels the EIP in the AWS Console for easy identification

**Resource Dependency:**
```
aws_instance.datacenter_ec2
        │
        │  (implicit dependency via instance ID reference)
        ▼
aws_eip.datacenter_eip
```
Terraform creates the EC2 instance **first**, then allocates and associates the Elastic IP.

**File Content (`main.tf`):**
```hcl
# EC2 INSTANCE
resource "aws_instance" "datacenter_ec2" {
  ami           = "ami-0c02fb55956c7d316" # Ubuntu Linux AMI
  instance_type = "t2.micro"

  tags = {
    Name = "datacenter-ec2"
  }
}

# ELASTIC IP
resource "aws_eip" "datacenter_eip" {
  instance = aws_instance.datacenter_ec2.id

  tags = {
    Name = "datacenter-eip"
  }
}
```

---

### Step 3: Create Outputs Configuration File

Create `outputs.tf` to export the instance name and Elastic IP address:

```bash
cat > outputs.tf << 'EOF'
output "KKE_instance_name" {
  description = "Name of the EC2 instance"
  value       = aws_instance.datacenter_ec2.tags["Name"]
}

output "KKE_eip" {
  description = "Elastic IP associated with the instance"
  value       = aws_eip.datacenter_eip.public_ip
}
EOF
```

**Purpose:** Export the EC2 instance name and the allocated Elastic IP address for verification and downstream reference.

**Configuration Breakdown:**
- `KKE_instance_name` — Extracts the `Name` tag from the EC2 instance resource
- `KKE_eip` — Extracts the `public_ip` attribute from the EIP resource (the static IPv4 address)
- **Best Practice:** Outputting the EIP address allows it to be used by other Terraform configurations, CI/CD pipelines, or DNS record automation

**File Content (`outputs.tf`):**
```hcl
output "KKE_instance_name" {
  description = "Name of the EC2 instance"
  value       = aws_instance.datacenter_ec2.tags["Name"]
}

output "KKE_eip" {
  description = "Elastic IP associated with the instance"
  value       = aws_eip.datacenter_eip.public_ip
}
```

---

### Step 4: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Purpose:** Confirm both required files are present before initializing.

**Expected Output:**
```
total 12
drwxr-xr-x 2 bob bob 4096 Feb 25 10:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 09:55 ..
-rw-r--r-- 1 bob bob  342 Feb 25 10:00 main.tf
-rw-r--r-- 1 bob bob  228 Feb 25 10:00 outputs.tf
```

**Success Indicators:**
- ✅ Both files present: `main.tf` and `outputs.tf`
- ✅ Files have appropriate read permissions
- ✅ Files are in the correct working directory

---

### Step 5: Initialize Terraform

```bash
terraform init
```

**Purpose:** Initialize the working directory, download the AWS provider, and prepare the backend.

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
- ✅ `.terraform.lock.hcl` generated

---

### Step 6: Validate Configuration

```bash
terraform validate
```

**Purpose:** Verify HCL syntax and internal consistency of both configuration files.

**Expected Output:**
```
Success! The configuration is valid.
```

**What's Validated:**
- ✅ `aws_instance` and `aws_eip` resource blocks are syntactically correct
- ✅ `instance = aws_instance.datacenter_ec2.id` is a valid attribute reference
- ✅ Output value references resolve correctly

---

### Step 7: Format Configuration Files

```bash
terraform fmt
```

**Purpose:** Auto-format all `.tf` files to canonical HCL style.

**Expected Output:**
```
main.tf
outputs.tf
```

**Formatting Applied:**
- ✅ Consistent 2-space indentation
- ✅ Proper alignment of `=` signs
- ✅ Consistent blank lines between resource blocks

---

### Step 8: Review Execution Plan

```bash
terraform plan
```

**Purpose:** Preview the changes Terraform will make — 1 EC2 instance and 1 Elastic IP.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_eip.datacenter_eip will be created
  + resource "aws_eip" "datacenter_eip" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags                 = {
          + "Name" = "datacenter-eip"
        }
      + vpc                  = (known after apply)
    }

  # aws_instance.datacenter_ec2 will be created
  + resource "aws_instance" "datacenter_ec2" {
      + ami                  = "ami-0c02fb55956c7d316"
      + arn                  = (known after apply)
      + id                   = (known after apply)
      + instance_state       = (known after apply)
      + instance_type        = "t2.micro"
      + private_ip           = (known after apply)
      + public_ip            = (known after apply)
      + tags                 = {
          + "Name" = "datacenter-ec2"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + KKE_eip           = (known after apply)
  + KKE_instance_name = "datacenter-ec2"

───────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
```

**Plan Summary:**
- **Resources to Create:** 2 (EC2 Instance + Elastic IP)
- **Resources to Change:** 0
- **Resources to Destroy:** 0
- **Outputs:** `KKE_instance_name` (known now) + `KKE_eip` (known after apply)

**Success Indicators:**
- ✅ 2 resources planned for creation
- ✅ `instance_type = "t2.micro"` confirmed
- ✅ `KKE_instance_name = "datacenter-ec2"` shown in plan
- ✅ `KKE_eip` will be revealed after apply (public IP assigned by AWS)

---

### Step 9: Apply Configuration

```bash
terraform apply -auto-approve
```

**Purpose:** Provision the EC2 instance and allocate + associate the Elastic IP in AWS.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.

Plan: 2 to add, 0 to change, 0 to destroy.

aws_instance.datacenter_ec2: Creating...
aws_instance.datacenter_ec2: Still creating... [10s elapsed]
aws_instance.datacenter_ec2: Still creating... [20s elapsed]
aws_instance.datacenter_ec2: Creation complete after 32s [id=i-0abc123def456789a]
aws_eip.datacenter_eip: Creating...
aws_eip.datacenter_eip: Creation complete after 1s [id=eipalloc-0a1b2c3d4e5f67890]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

KKE_eip           = "54.123.45.67"
KKE_instance_name = "datacenter-ec2"
```

**Success Indicators:**
- ✅ EC2 instance created first (implicit dependency honored)
- ✅ Elastic IP allocated and associated immediately after
- ✅ `KKE_eip` displays the static public IP address
- ✅ `KKE_instance_name` displays `"datacenter-ec2"`
- ✅ `2 added, 0 changed, 0 destroyed`

**Creation Order Verification:**
| Order | Resource | Reason |
|-------|----------|--------|
| 1st | `aws_instance.datacenter_ec2` | No dependencies |
| 2nd | `aws_eip.datacenter_eip` | Needs instance ID |

---

### Step 10: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Purpose:** Confirm infrastructure matches configuration with zero pending changes.

**Expected Output:**
```
aws_instance.datacenter_ec2: Refreshing state... [id=i-0abc123def456789a]
aws_eip.datacenter_eip: Refreshing state... [id=eipalloc-0a1b2c3d4e5f67890]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ Both resources refreshed from live AWS state
- ✅ No drift between Terraform state and actual AWS resources
- ✅ Task is ready for submission

**⚠️ Important:** This "No changes" verification is **required before task submission**.

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Expected Output:**
```
aws_eip.datacenter_eip
aws_instance.datacenter_ec2
```

**Success Indicators:**
- ✅ Both resources tracked in state
- ✅ No unexpected resources

---

### Step 2: Inspect EC2 Instance Details

```bash
terraform state show aws_instance.datacenter_ec2
```

**Expected Output:**
```
# aws_instance.datacenter_ec2:
resource "aws_instance" "datacenter_ec2" {
    ami                    = "ami-0c02fb55956c7d316"
    id                     = "i-0abc123def456789a"
    instance_type          = "t2.micro"
    public_ip              = "54.123.45.67"
    tags                   = {
        "Name" = "datacenter-ec2"
    }
}
```

**Verification Points:**
- ✅ `instance_type = "t2.micro"`
- ✅ `tags.Name = "datacenter-ec2"`
- ✅ `public_ip` matches the Elastic IP

---

### Step 3: Inspect Elastic IP Details

```bash
terraform state show aws_eip.datacenter_eip
```

**Expected Output:**
```
# aws_eip.datacenter_eip:
resource "aws_eip" "datacenter_eip" {
    allocation_id = "eipalloc-0a1b2c3d4e5f67890"
    association_id = "eipassoc-0z9y8x7w6v5u43210"
    domain        = "vpc"
    id            = "eipalloc-0a1b2c3d4e5f67890"
    instance      = "i-0abc123def456789a"
    public_ip     = "54.123.45.67"
    tags          = {
        "Name" = "datacenter-eip"
    }
    vpc           = true
}
```

**Verification Points:**
- ✅ `public_ip` is populated with a static IPv4 address
- ✅ `instance` field references the correct EC2 instance ID
- ✅ `tags.Name = "datacenter-eip"`
- ✅ `association_id` confirms the EIP is actively associated with the instance

---

### Step 4: View Output Values

```bash
terraform output
```

**Expected Output:**
```
KKE_eip           = "54.123.45.67"
KKE_instance_name = "datacenter-ec2"
```

**Individual Output Access:**
```bash
terraform output KKE_instance_name
terraform output KKE_eip
```

**Expected Output:**
```
"datacenter-ec2"
"54.123.45.67"
```

---

### Step 5: AWS CLI Verification (Optional)

```bash
# Verify EC2 instance
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].{ID:InstanceId,State:State.Name,Type:InstanceType,PublicIP:PublicIpAddress}" \
  --output table

# Verify Elastic IP allocation and association
aws ec2 describe-addresses \
  --filters "Name=tag:Name,Values=datacenter-eip" \
  --query "Addresses[].{EIP:PublicIp,InstanceId:InstanceId,AllocationId:AllocationId}" \
  --output table
```

---

## 🔍 Code Analysis

### Resource Relationship Diagram

```
aws_instance.datacenter_ec2
        │  id = "i-0abc123..."
        │
        ▼  (aws_eip.instance references this ID)
aws_eip.datacenter_eip
        │  public_ip = "54.123.45.67"  (static)
        │
        ▼
  Application Access Point
  (DNS record, firewall rule, etc. — all point to 54.123.45.67)
```

### Elastic IP vs Default Public IP

```hcl
# WITHOUT Elastic IP — dynamic public IP
resource "aws_instance" "example" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  # public_ip changes every stop/start cycle ❌
}

# WITH Elastic IP — static public IP (this task)
resource "aws_instance" "datacenter_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
}

resource "aws_eip" "datacenter_eip" {
  instance = aws_instance.datacenter_ec2.id
  # public_ip is permanent until EIP is released ✅
}
```

### Output Variable Analysis

| Output | Source Attribute | Value Type | Example Value |
|--------|-----------------|------------|---------------|
| `KKE_instance_name` | `aws_instance.datacenter_ec2.tags["Name"]` | string | `"datacenter-ec2"` |
| `KKE_eip` | `aws_eip.datacenter_eip.public_ip` | string | `"54.123.45.67"` |

### Key `aws_eip` Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `public_ip` | The static IPv4 address | `54.123.45.67` |
| `allocation_id` | Unique ID of the EIP allocation | `eipalloc-0a1b...` |
| `association_id` | ID confirming EIP is linked to an instance | `eipassoc-0z9y...` |
| `instance` | ID of the associated EC2 instance | `i-0abc123...` |
| `domain` | Scope — `vpc` for VPC instances | `vpc` |

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate syntax
terraform validate

# 2. Preview plan — expect 2 resources to create
terraform plan

# 3. Apply configuration
terraform apply -auto-approve

# 4. List resources in state
terraform state list

# 5. Confirm EIP is associated
terraform state show aws_eip.datacenter_eip

# 6. View outputs
terraform output

# 7. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| EC2 instance name | `datacenter-ec2` | `terraform output KKE_instance_name` |
| Instance type | `t2.micro` | `terraform state show aws_instance.datacenter_ec2` |
| EIP public IP | Static IPv4 (e.g., `54.x.x.x`) | `terraform output KKE_eip` |
| EIP association | Instance ID populated | `terraform state show aws_eip.datacenter_eip` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: AMI Not Found

**Error:**
```
Error: creating EC2 Instance: InvalidAMIID.NotFound: The image id 'ami-0c02fb55956c7d316' does not exist
```

**Cause:** The AMI ID is region-specific. It may not exist in the region your AWS provider is configured for.

**Solution:**
```bash
# Find a valid Ubuntu AMI for your region
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*" \
            "Name=state,Values=available" \
  --query "Images | sort_by(@, &CreationDate) | [-1].ImageId" \
  --output text

# Update main.tf with the returned AMI ID
```

---

### Issue 2: Elastic IP Address Limit Exceeded

**Error:**
```
Error: creating EC2 EIP: AddressLimitExceeded: The maximum number of addresses has been reached.
```

**Cause:** AWS accounts are limited to **5 Elastic IPs per region** by default.

**Solution:**
```bash
# List existing EIPs in your account
aws ec2 describe-addresses --query "Addresses[].{IP:PublicIp,ID:AllocationId,Instance:InstanceId}" --output table

# Release any unused EIPs (those without an InstanceId)
aws ec2 release-address --allocation-id eipalloc-0EXAMPLE

# Or request a limit increase from AWS Support
```

---

### Issue 3: EIP Not Associated with Instance

**Symptom:** `terraform state show aws_eip.datacenter_eip` shows `association_id = null`.

**Cause:** The `instance` attribute in `aws_eip` may be missing or referencing an incorrect resource.

**Solution:** Verify `main.tf` contains:
```hcl
resource "aws_eip" "datacenter_eip" {
  instance = aws_instance.datacenter_ec2.id   # Must reference .id
  # ...
}
```

Then re-apply:
```bash
terraform apply -auto-approve
```

---

### Issue 4: EIP Costs — Instance Must Be Running

**Warning:** AWS charges for Elastic IPs that are **allocated but not associated** with a running instance.

**Best Practice:**
```bash
# Always destroy test infrastructure when not in use
terraform destroy -auto-approve

# This releases the EIP and terminates the EC2 instance
# No ongoing charges after destroy
```

---

### Issue 5: Public IP Not Showing in Output

**Symptom:** `KKE_eip` output is empty or `null`.

**Cause:** The EIP may not have been fully associated at the time of the output evaluation.

**Solution:**
```bash
# Refresh state to pull latest values from AWS
terraform refresh

# Then view outputs
terraform output
```

---

## 📚 Best Practices

### 1. Always Use Elastic IPs for Production Instances
Default EC2 public IPs are ephemeral — they change on every stop/start. Any DNS records, SSL certificates, or firewall rules pointing to the old IP break. Elastic IPs prevent this volatility.

### 2. Tag Elastic IPs for Cost Tracking
```hcl
tags = {
  Name        = "datacenter-eip"
  Environment = "production"
  Owner       = "devops-team"
}
```
Untagged EIPs are difficult to track, especially when orphaned (allocated but not associated).

### 3. Output the EIP for Downstream Use
```hcl
output "KKE_eip" {
  value = aws_eip.datacenter_eip.public_ip
}
```
Outputting the EIP address allows it to be consumed by other Terraform workspaces, DNS automation scripts, or CI/CD pipelines.

### 4. Be Aware of EIP Billing
AWS charges for Elastic IPs that are:
- Allocated but **not associated** with a running instance
- Associated with a **stopped** instance

Always destroy unused EIPs to avoid unexpected charges.

### 5. Use Implicit Dependencies (No `depends_on` Needed)
The `instance = aws_instance.datacenter_ec2.id` line in `aws_eip` automatically creates a dependency. Terraform knows to create the EC2 instance first. **No explicit `depends_on` is required.**

### 6. Use `vpc` Domain for VPC-Based Instances
If your EC2 instance is in a VPC (which is default in modern AWS), the EIP is automatically in the `vpc` domain. For very old EC2-Classic instances (deprecated), this would need to be `ec2`.

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Name | Configuration |
|----------|------|---------------|
| **EC2 Instance** | `datacenter-ec2` | `t2.micro`, Ubuntu AMI |
| **Elastic IP** | `datacenter-eip` | Static public IPv4, associated with instance |

### Files Created

| File | Purpose |
|------|---------|
| `main.tf` | EC2 instance + EIP resource definitions |
| `outputs.tf` | Exports `KKE_instance_name` and `KKE_eip` |

### Task Completion Checklist

- [x] ✅ EC2 instance `datacenter-ec2` created with Ubuntu AMI and `t2.micro` type
- [x] ✅ Elastic IP `datacenter-eip` allocated and associated with the instance
- [x] ✅ All resources defined in **single `main.tf`** file
- [x] ✅ `outputs.tf` defines `KKE_instance_name` (instance name tag)
- [x] ✅ `outputs.tf` defines `KKE_eip` (static public IP address)
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply -auto-approve` — **2 resources added**
- [x] ✅ Elastic IP output shows a valid static public IPv4 address
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task demonstrates how to pair an **AWS EC2 instance with an Elastic IP** using Terraform — a critical pattern for production infrastructure where a **stable, persistent public IP address** is required. The `aws_eip` resource's `instance` attribute creates an automatic implicit dependency, ensuring correct provisioning order. The Elastic IP persists through instance stop/start cycles, making it ideal for hosting applications where DNS records, SSL certificates, or external firewall rules depend on a fixed endpoint.
