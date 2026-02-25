# 🌟 Task 04 - Provision Multiple EC2 Instances Using Terraform `count` Parameter

## 📌 Task Description

The **Nautilus DevOps team** wants to provision multiple EC2 instances in AWS using Terraform in a **modular and scalable** manner. Instead of defining each instance separately, Terraform's `count` parameter enables the deployment of several identically configured instances with a consistent naming convention — a foundational pattern for scalable infrastructure management.

**Requirements:**
- Create **3 EC2 instances** using the `count` parameter
- Name each instance with the prefix **`xfusion-instance`** (e.g., `xfusion-instance-1`, `xfusion-instance-2`, `xfusion-instance-3`)
- Instance type must be **`t2.micro`**
- Key pair name must be **`xfusion-key`**
- All resources must be defined in a single **`main.tf`** file
- Use a **`variables.tf`** file with the following variables:
  - `KKE_INSTANCE_COUNT` — number of instances
  - `KKE_INSTANCE_TYPE` — instance type
  - `KKE_KEY_NAME` — key pair name
  - `KKE_INSTANCE_PREFIX` — instance name prefix
- Use a **`locals.tf`** file to define a local `AMI_ID` that retrieves the **latest Amazon Linux 2 AMI** via a data source
- Use **`terraform.tfvars`** to assign values to all variables
- Use an **`outputs.tf`** file to output:
  - `kke_instance_names` — list of all created instance names
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Use Terraform's `count` meta-argument, a data source for dynamic AMI lookup, and locals to provision a scalable fleet of identically configured EC2 instances.

💡 **Note:** Using `count` with a `count.index`-based naming convention is a standard pattern for homogeneous instance fleets. The `data "aws_ami"` source ensures the latest Amazon Linux 2 AMI is always used without hardcoding AMI IDs.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- 3 × AWS EC2 Instances (`t2.micro`) with consistent naming
- Data source for dynamic Amazon Linux 2 AMI lookup
- Local variable for AMI ID resolution

**Working Directory:** `/home/bob/terraform`

**Instance Configuration:**
| Instance | Name Tag | Type | Key Pair | AMI |
|----------|----------|------|----------|-----|
| Instance 1 | `xfusion-instance-1` | `t2.micro` | `xfusion-key` | Latest Amazon Linux 2 |
| Instance 2 | `xfusion-instance-2` | `t2.micro` | `xfusion-key` | Latest Amazon Linux 2 |
| Instance 3 | `xfusion-instance-3` | `t2.micro` | `xfusion-key` | Latest Amazon Linux 2 |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`count` Meta-Argument:** Tells Terraform to create `N` copies of the resource block
- **`count.index`:** Zero-based counter used to generate unique names (`count.index + 1` → 1, 2, 3)
- **Data Source (`aws_ami`):** Dynamically queries AWS for the most recent Amazon Linux 2 AMI
- **Local Variable (`AMI_ID`):** Stores the resolved AMI ID from the data source for clean reference in `main.tf`
- **`for` Expression in Output:** Iterates over all created instances to collect their name tags

### 🎯 Implementation Strategy
1. Define variables in `variables.tf`
2. Assign values in `terraform.tfvars`
3. Create `locals.tf` with a data source and local AMI ID
4. Create `main.tf` using `count` to provision 3 instances
5. Define `outputs.tf` to list all instance names
6. Initialize, validate, format, plan, apply, and verify

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf           # EC2 instance resource using count
├── variables.tf      # Variable declarations
├── terraform.tfvars  # Variable value assignments
├── locals.tf         # Data source + local AMI_ID variable
└── outputs.tf        # Output: list of instance names
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

### Step 2: Create Variables Definition File

Create `variables.tf` to declare all input variables:

```bash
cat > variables.tf << 'EOF'
variable "KKE_INSTANCE_COUNT" {
  description = "Number of EC2 instances"
  type        = number
}

variable "KKE_INSTANCE_TYPE" {
  description = "EC2 instance type"
  type        = string
}

variable "KKE_KEY_NAME" {
  description = "SSH key pair name"
  type        = string
}

variable "KKE_INSTANCE_PREFIX" {
  description = "Prefix for EC2 instance naming"
  type        = string
}
EOF
```

**Purpose:** Declare all configurable parameters as typed variables for flexibility and reusability.

**Configuration Breakdown:**
- `KKE_INSTANCE_COUNT` — `number` type; controls how many instances `count` creates
- `KKE_INSTANCE_TYPE` — `string` type; EC2 instance size (e.g., `t2.micro`)
- `KKE_KEY_NAME` — `string` type; name of the pre-existing AWS key pair
- `KKE_INSTANCE_PREFIX` — `string` type; base prefix for generating unique instance names
- **Best Practice:** Declaring all configurable values as variables avoids hardcoding and enables environment-specific deployments

**File Content (`variables.tf`):**
```hcl
variable "KKE_INSTANCE_COUNT" {
  description = "Number of EC2 instances"
  type        = number
}

variable "KKE_INSTANCE_TYPE" {
  description = "EC2 instance type"
  type        = string
}

variable "KKE_KEY_NAME" {
  description = "SSH key pair name"
  type        = string
}

variable "KKE_INSTANCE_PREFIX" {
  description = "Prefix for EC2 instance naming"
  type        = string
}
```

---

### Step 3: Create Variable Values File

Create `terraform.tfvars` to assign concrete values to all declared variables:

```bash
cat > terraform.tfvars << 'EOF'
KKE_INSTANCE_COUNT  = 3
KKE_INSTANCE_TYPE   = "t2.micro"
KKE_KEY_NAME        = "xfusion-key"
KKE_INSTANCE_PREFIX = "xfusion-instance"
EOF
```

**Purpose:** Provide the actual runtime values that drive the infrastructure configuration.

**Configuration Breakdown:**
- `KKE_INSTANCE_COUNT = 3` — Creates exactly 3 EC2 instances via `count`
- `KKE_INSTANCE_TYPE = "t2.micro"` — Cost-effective general-purpose instance size
- `KKE_KEY_NAME = "xfusion-key"` — Pre-existing EC2 key pair for SSH access
- `KKE_INSTANCE_PREFIX = "xfusion-instance"` — Base string for name tags: results in `xfusion-instance-1`, `-2`, `-3`

**File Content (`terraform.tfvars`):**
```hcl
KKE_INSTANCE_COUNT  = 3
KKE_INSTANCE_TYPE   = "t2.micro"
KKE_KEY_NAME        = "xfusion-key"
KKE_INSTANCE_PREFIX = "xfusion-instance"
```

---

### Step 4: Create Locals and Data Source File

Create `locals.tf` with the Amazon Linux 2 data source and local AMI variable:

```bash
cat > locals.tf << 'EOF'
# LOCALS + DATA SOURCE FOR LATEST AMAZON LINUX 2 AMI

data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

locals {
  AMI_ID = data.aws_ami.amazon_linux_2.id
}
EOF
```

**Purpose:** Dynamically resolve the latest Amazon Linux 2 AMI ID at plan time, avoiding hardcoded AMI values that become stale across AWS regions and time.

**Configuration Breakdown:**

**Data Source (`data "aws_ami" "amazon_linux_2"`):**
- `most_recent = true` — Selects the newest AMI matching the filters
- `owners = ["amazon"]` — Restricts to official Amazon-owned AMIs (prevents using unofficial/third-party AMIs)
- `filter { name = "name", values = ["amzn2-ami-hvm-*-x86_64-gp2"] }` — Pattern match for Amazon Linux 2 HVM AMIs using GP2 storage on x86_64 architecture

**Local Variable (`locals { AMI_ID }`):**
- `AMI_ID = data.aws_ami.amazon_linux_2.id` — Stores the resolved AMI ID into a local for clean reference in `main.tf` as `local.AMI_ID`
- **Best Practice:** Using a local to alias the data source output improves readability and makes it easy to swap AMI sources in one place

**AMI Filter Explained:**
| Filter Component | Value | Meaning |
|-----------------|-------|---------|
| `amzn2-ami-hvm-*` | Wildcard prefix | Amazon Linux 2 HVM virtualization |
| `x86_64` | Architecture | 64-bit Intel/AMD |
| `gp2` | Storage type | General Purpose SSD |

**File Content (`locals.tf`):**
```hcl
# LOCALS + DATA SOURCE FOR LATEST AMAZON LINUX 2 AMI

data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

locals {
  AMI_ID = data.aws_ami.amazon_linux_2.id
}
```

---

### Step 5: Create Main Configuration File

Create `main.tf` with the EC2 instance resource using `count`:

```bash
cat > main.tf << 'EOF'
# MAIN FILE TO CREATE MULTIPLE EC2 INSTANCES

resource "aws_instance" "xfusion_ec2" {
  count         = var.KKE_INSTANCE_COUNT
  ami           = local.AMI_ID
  instance_type = var.KKE_INSTANCE_TYPE
  key_name      = var.KKE_KEY_NAME

  tags = {
    Name = "${var.KKE_INSTANCE_PREFIX}-${count.index + 1}"
  }
}
EOF
```

**Purpose:** Define a single EC2 resource block that Terraform replicates `KKE_INSTANCE_COUNT` times, each with a unique name tag derived from `count.index`.

**Configuration Breakdown:**

- `count = var.KKE_INSTANCE_COUNT` — Terraform creates this many instance copies (3 in this task)
- `ami = local.AMI_ID` — References the dynamically resolved AMI from `locals.tf` — always current, never stale
- `instance_type = var.KKE_INSTANCE_TYPE` — Sourced from variable: `t2.micro`
- `key_name = var.KKE_KEY_NAME` — References the pre-existing `xfusion-key` key pair
- `Name = "${var.KKE_INSTANCE_PREFIX}-${count.index + 1}"` — Generates unique names:
  - `count.index = 0` → `xfusion-instance-1`
  - `count.index = 1` → `xfusion-instance-2`
  - `count.index = 2` → `xfusion-instance-3`

**How `count.index` Generates Names:**
```
count = 3  →  Terraform creates 3 resource instances:

  aws_instance.xfusion_ec2[0]  →  Name: "xfusion-instance-1"  (0 + 1)
  aws_instance.xfusion_ec2[1]  →  Name: "xfusion-instance-2"  (1 + 1)
  aws_instance.xfusion_ec2[2]  →  Name: "xfusion-instance-3"  (2 + 1)
```

**File Content (`main.tf`):**
```hcl
# MAIN FILE TO CREATE MULTIPLE EC2 INSTANCES

resource "aws_instance" "xfusion_ec2" {
  count         = var.KKE_INSTANCE_COUNT
  ami           = local.AMI_ID
  instance_type = var.KKE_INSTANCE_TYPE
  key_name      = var.KKE_KEY_NAME

  tags = {
    Name = "${var.KKE_INSTANCE_PREFIX}-${count.index + 1}"
  }
}
```

---

### Step 6: Create Outputs Configuration File

Create `outputs.tf` to export the list of all instance names:

```bash
cat > outputs.tf << 'EOF'
output "kke_instance_names" {
  description = "List of EC2 instance names"
  value       = [for inst in aws_instance.xfusion_ec2 : inst.tags.Name]
}
EOF
```

**Purpose:** Export a list of all provisioned instance names using a `for` expression, enabling easy verification and downstream reference.

**Configuration Breakdown:**
- `[for inst in aws_instance.xfusion_ec2 : inst.tags.Name]` — Iterates over the list of all created instances and collects each `Name` tag
- When `count` is used, `aws_instance.xfusion_ec2` becomes a **list** of objects — the `for` expression efficiently processes this list
- **Best Practice:** Using `for` in outputs is the correct way to aggregate attributes from `count`-based resources

**File Content (`outputs.tf`):**
```hcl
output "kke_instance_names" {
  description = "List of EC2 instance names"
  value       = [for inst in aws_instance.xfusion_ec2 : inst.tags.Name]
}
```

---

### Step 7: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Purpose:** Confirm all five required files are present before initializing.

**Expected Output:**
```
total 20
drwxr-xr-x 2 bob bob 4096 Feb 25 10:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 09:55 ..
-rw-r--r-- 1 bob bob  320 Feb 25 10:00 locals.tf
-rw-r--r-- 1 bob bob  245 Feb 25 10:00 main.tf
-rw-r--r-- 1 bob bob  148 Feb 25 10:00 outputs.tf
-rw-r--r-- 1 bob bob   96 Feb 25 10:00 terraform.tfvars
-rw-r--r-- 1 bob bob  310 Feb 25 10:00 variables.tf
```

**Success Indicators:**
- ✅ All five files present: `main.tf`, `variables.tf`, `terraform.tfvars`, `locals.tf`, `outputs.tf`
- ✅ Files have appropriate read permissions
- ✅ All files in the correct working directory

---

### Step 8: Initialize Terraform

```bash
terraform init
```

**Purpose:** Initialize the directory, download the AWS provider, and prepare the backend.

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

### Step 9: Validate Configuration

```bash
terraform validate
```

**Purpose:** Verify syntax and configuration consistency across all five files.

**Expected Output:**
```
Success! The configuration is valid.
```

**What's Validated:**
- ✅ `count` usage is valid in the resource block
- ✅ `local.AMI_ID` correctly references the data source in `locals.tf`
- ✅ `count.index` is used only within a counted resource block
- ✅ `for` expression in `outputs.tf` is syntactically correct
- ✅ All variable references resolve to declared variables

---

### Step 10: Format Configuration Files

```bash
terraform fmt
```

**Purpose:** Automatically format all `.tf` files to canonical HCL style.

**Expected Output:**
```
locals.tf
main.tf
outputs.tf
terraform.tfvars
variables.tf
```

**Formatting Applied:**
- ✅ Consistent 2-space indentation across all files
- ✅ Proper alignment of `=` signs in attribute blocks
- ✅ Standardized spacing around `for` expressions

---

### Step 11: Review Execution Plan

```bash
terraform plan
```

**Purpose:** Preview all changes Terraform will make — 3 new EC2 instances with unique names.

**Expected Output:**
```
data.aws_ami.amazon_linux_2: Reading...
data.aws_ami.amazon_linux_2: Read complete after 1s [id=ami-0abcdef1234567890]

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.xfusion_ec2[0] will be created
  + resource "aws_instance" "xfusion_ec2" {
      + ami           = "ami-0abcdef1234567890"
      + id            = (known after apply)
      + instance_type = "t2.micro"
      + key_name      = "xfusion-key"
      + tags          = {
          + "Name" = "xfusion-instance-1"
        }
    }

  # aws_instance.xfusion_ec2[1] will be created
  + resource "aws_instance" "xfusion_ec2" {
      + ami           = "ami-0abcdef1234567890"
      + id            = (known after apply)
      + instance_type = "t2.micro"
      + key_name      = "xfusion-key"
      + tags          = {
          + "Name" = "xfusion-instance-2"
        }
    }

  # aws_instance.xfusion_ec2[2] will be created
  + resource "aws_instance" "xfusion_ec2" {
      + ami           = "ami-0abcdef1234567890"
      + id            = (known after apply)
      + instance_type = "t2.micro"
      + key_name      = "xfusion-key"
      + tags          = {
          + "Name" = "xfusion-instance-3"
        }
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + kke_instance_names = [
      + "xfusion-instance-1",
      + "xfusion-instance-2",
      + "xfusion-instance-3",
    ]

───────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
```

**Plan Summary:**
- **Resources to Create:** 3 (`xfusion_ec2[0]`, `[1]`, `[2]`)
- **Resources to Change:** 0
- **Resources to Destroy:** 0
- **Data Sources Read:** 1 (`aws_ami.amazon_linux_2`)
- **Outputs to Create:** 1 (list of 3 names)

**Success Indicators:**
- ✅ Data source resolved AMI ID during plan
- ✅ All 3 instances planned with unique names (`-1`, `-2`, `-3`)
- ✅ All instances use `t2.micro` and `xfusion-key`
- ✅ Output shows correct list of names

---

### Step 12: Apply Configuration

```bash
terraform apply -auto-approve
```

**Purpose:** Provision all 3 EC2 instances in AWS simultaneously.

**Expected Output:**
```
data.aws_ami.amazon_linux_2: Reading...
data.aws_ami.amazon_linux_2: Read complete after 1s [id=ami-0abcdef1234567890]

Terraform used the selected providers to generate the following execution plan.

Plan: 3 to add, 0 to change, 0 to destroy.

aws_instance.xfusion_ec2[0]: Creating...
aws_instance.xfusion_ec2[1]: Creating...
aws_instance.xfusion_ec2[2]: Creating...
aws_instance.xfusion_ec2[0]: Still creating... [10s elapsed]
aws_instance.xfusion_ec2[1]: Still creating... [10s elapsed]
aws_instance.xfusion_ec2[2]: Still creating... [10s elapsed]
aws_instance.xfusion_ec2[0]: Creation complete after 32s [id=i-0aaa111bbb222ccc0]
aws_instance.xfusion_ec2[1]: Creation complete after 33s [id=i-0aaa111bbb222ccc1]
aws_instance.xfusion_ec2[2]: Creation complete after 33s [id=i-0aaa111bbb222ccc2]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

kke_instance_names = [
  "xfusion-instance-1",
  "xfusion-instance-2",
  "xfusion-instance-3",
]
```

**Success Indicators:**
- ✅ All 3 instances created **in parallel** (`count`-based resources are created concurrently)
- ✅ Each instance has a unique AWS Instance ID
- ✅ Output `kke_instance_names` lists all 3 names correctly
- ✅ `3 added, 0 changed, 0 destroyed`

---

### Step 13: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Purpose:** Confirm all provisioned instances match the configuration with zero pending changes.

**Expected Output:**
```
data.aws_ami.amazon_linux_2: Reading...
data.aws_ami.amazon_linux_2: Read complete after 1s [id=ami-0abcdef1234567890]
aws_instance.xfusion_ec2[0]: Refreshing state... [id=i-0aaa111bbb222ccc0]
aws_instance.xfusion_ec2[1]: Refreshing state... [id=i-0aaa111bbb222ccc1]
aws_instance.xfusion_ec2[2]: Refreshing state... [id=i-0aaa111bbb222ccc2]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ All 3 instances refreshed from AWS state
- ✅ Data source re-read and still resolves the same AMI ID
- ✅ No configuration drift — task is ready for submission

**⚠️ Important:** This step is **required before task submission**.

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Purpose:** Confirm all instances and the data source are tracked in state.

**Expected Output:**
```
data.aws_ami.amazon_linux_2
aws_instance.xfusion_ec2[0]
aws_instance.xfusion_ec2[1]
aws_instance.xfusion_ec2[2]
```

**Success Indicators:**
- ✅ Data source listed (read during plan/apply)
- ✅ All 3 instances listed with `[index]` notation
- ✅ No unexpected resources in state

---

### Step 2: Inspect Individual Instance Details

```bash
terraform state show 'aws_instance.xfusion_ec2[0]'
terraform state show 'aws_instance.xfusion_ec2[1]'
terraform state show 'aws_instance.xfusion_ec2[2]'
```

**Purpose:** Confirm each instance has the correct configuration.

**Expected Output (for instance[0]):**
```
# aws_instance.xfusion_ec2[0]:
resource "aws_instance" "xfusion_ec2" {
    ami           = "ami-0abcdef1234567890"
    id            = "i-0aaa111bbb222ccc0"
    instance_type = "t2.micro"
    key_name      = "xfusion-key"
    tags          = {
        "Name" = "xfusion-instance-1"
    }
}
```

**Verification Points:**
- ✅ `instance_type = "t2.micro"`
- ✅ `key_name = "xfusion-key"`
- ✅ `tags.Name` follows `xfusion-instance-{N}` pattern
- ✅ `ami` matches latest Amazon Linux 2 AMI

---

### Step 3: Verify Output Values

```bash
terraform output
```

**Purpose:** Display the list of all instance names exported by `outputs.tf`.

**Expected Output:**
```
kke_instance_names = [
  "xfusion-instance-1",
  "xfusion-instance-2",
  "xfusion-instance-3",
]
```

**Direct Output Access:**
```bash
terraform output kke_instance_names
```

---

### Step 4: AWS CLI Verification (Optional)

```bash
# List all xfusion-instance EC2 instances
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-instance-*" \
            "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].{ID:InstanceId,Name:Tags[?Key=='Name']|[0].Value,Type:InstanceType}" \
  --output table
```

**Expected Output:**
```
-------------------------------------------------------------
|               DescribeInstances                           |
+----------------------+---------------------+--------------+
|          ID          |        Name         |    Type      |
+----------------------+---------------------+--------------+
|  i-0aaa111bbb222ccc0 |  xfusion-instance-1 |  t2.micro   |
|  i-0aaa111bbb222ccc1 |  xfusion-instance-2 |  t2.micro   |
|  i-0aaa111bbb222ccc2 |  xfusion-instance-3 |  t2.micro   |
+----------------------+---------------------+--------------+
```

---

## 🔍 Code Analysis

### Complete Solution Structure

```bash
/home/bob/terraform/
├── main.tf           # EC2 resource block with count meta-argument
├── variables.tf      # 4 variable declarations
├── terraform.tfvars  # Variable value assignments
├── locals.tf         # Data source + AMI_ID local
└── outputs.tf        # kke_instance_names list output
```

### `count` Meta-Argument Deep Dive

```hcl
resource "aws_instance" "xfusion_ec2" {
  count = var.KKE_INSTANCE_COUNT   # 3

  # count.index = 0, 1, 2 (zero-based)
  tags = {
    Name = "${var.KKE_INSTANCE_PREFIX}-${count.index + 1}"
    #         "xfusion-instance"    -   1, 2, 3
  }
}
```

**Resource Addressing with `count`:**
| Address | Name Tag | Index |
|---------|----------|-------|
| `aws_instance.xfusion_ec2[0]` | `xfusion-instance-1` | `count.index = 0` |
| `aws_instance.xfusion_ec2[1]` | `xfusion-instance-2` | `count.index = 1` |
| `aws_instance.xfusion_ec2[2]` | `xfusion-instance-3` | `count.index = 2` |

### Data Source + Local Variable Flow

```
data "aws_ami" "amazon_linux_2"         (in locals.tf)
      │   most_recent = true
      │   owners = ["amazon"]
      │   filter: amzn2-ami-hvm-*-x86_64-gp2
      │
      ▼
locals { AMI_ID = data.aws_ami.amazon_linux_2.id }
      │   (e.g., ami-0abcdef1234567890)
      │
      ▼
resource "aws_instance" "xfusion_ec2" {
  ami = local.AMI_ID              (in main.tf)
}
```

### Output `for` Expression Explained

```hcl
# aws_instance.xfusion_ec2 is a LIST when count is used:
# [
#   { id = "i-0...0", tags = { Name = "xfusion-instance-1" }, ... },
#   { id = "i-0...1", tags = { Name = "xfusion-instance-2" }, ... },
#   { id = "i-0...2", tags = { Name = "xfusion-instance-3" }, ... },
# ]

output "kke_instance_names" {
  value = [for inst in aws_instance.xfusion_ec2 : inst.tags.Name]
  # Result: ["xfusion-instance-1", "xfusion-instance-2", "xfusion-instance-3"]
}
```

### `count` vs `for_each` — When to Use Each

| Feature | `count` | `for_each` |
|---------|---------|-----------|
| **Best for** | Identical, homogeneous resources | Resources with distinct configurations |
| **Index type** | Integer (`[0]`, `[1]`, `[2]`) | String key (`["instance-a"]`) |
| **Resource address** | `resource[0]`, `resource[1]` | `resource["key"]` |
| **Removal behavior** | Renumbers all subsequent indexes | Only removes the specific keyed item |
| **This task** | ✅ Correct choice (identical instances) | Unnecessary for identical configs |

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate syntax and configuration
terraform validate

# 2. Preview the plan — expect 3 resources to create
terraform plan

# 3. Apply configuration
terraform apply -auto-approve

# 4. Verify all 3 instances in state
terraform state list

# 5. Check output values
terraform output kke_instance_names

# 6. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| Data source resolves AMI | AMI ID string (e.g., `ami-0abc...`) | `terraform plan` output |
| Instance count | 3 resources created | `terraform state list` |
| Instance names | `xfusion-instance-1/2/3` | `terraform output` |
| Instance type | `t2.micro` | `terraform state show` |
| Key pair | `xfusion-key` | `terraform state show` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: No Amazon Linux 2 AMI Found

**Error:**
```
Error: Your query returned no results. Please change your search criteria and try again.
```

**Cause:** The AMI filter pattern may not match any AMIs in the configured AWS region, or the region doesn't have Amazon Linux 2 AMIs.

**Solution:**
```bash
# Search for available AMIs in your region
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" \
  --query "Images | sort_by(@, &CreationDate) | [-1].{Name:Name, ID:ImageId}" \
  --output table

# If no results, try gp3 or a different architecture
# Update locals.tf filter accordingly
```

---

### Issue 2: Key Pair Does Not Exist

**Error:**
```
Error: creating EC2 Instance: InvalidKeyPair.NotFound: The key pair 'xfusion-key' does not exist
```

**Cause:** The `xfusion-key` key pair has not been created in AWS for this region.

**Solution:**
```bash
# Verify key pair exists in AWS
aws ec2 describe-key-pairs --key-names xfusion-key

# If it doesn't exist, create it
aws ec2 create-key-pair --key-name xfusion-key --query 'KeyMaterial' --output text > xfusion-key.pem
chmod 400 xfusion-key.pem
```

---

### Issue 3: Instance Count Mismatch

**Symptom:** Fewer or more than 3 instances created.

**Cause:** `KKE_INSTANCE_COUNT` value in `terraform.tfvars` is not set to `3`.

**Solution:**
```bash
# Check the tfvars file
cat terraform.tfvars

# Ensure it contains:
# KKE_INSTANCE_COUNT = 3

# Re-apply if corrected
terraform apply -auto-approve
```

---

### Issue 4: Incorrect Instance Names

**Symptom:** Instances named `xfusion-instance-0` to `xfusion-instance-2` instead of `1` to `3`.

**Cause:** `count.index + 1` was not applied — using `count.index` directly starts from `0`.

**Solution:** Ensure `main.tf` uses:
```hcl
Name = "${var.KKE_INSTANCE_PREFIX}-${count.index + 1}"
#                                              ^^^^ +1 required
```

---

### Issue 5: Plan Shows Changes After Apply (AMI Update)

**Symptom:**
```
Plan: 0 to add, 3 to change, 0 to destroy.
  ~ ami = "ami-0abcdef..." -> "ami-0newversion..."
```

**Cause:** The `most_recent = true` filter in the data source resolved a **newer AMI** than the one used during the original `apply`. This is expected behavior when Amazon publishes a new Amazon Linux 2 image.

**Solution:**
```bash
# Re-apply to update instances to the new AMI (if desired)
terraform apply -auto-approve

# OR pin to a specific AMI ID in locals.tf to prevent drift:
locals {
  AMI_ID = "ami-0abcdef1234567890"  # Pinned version
}
```

---

## 📚 Best Practices

### 1. Use `count` for Homogeneous Instances
When creating multiple identical copies of a resource, `count` is the right tool. For resources that need individual names or configurations, prefer `for_each` with a set or map.

### 2. Never Hardcode AMI IDs
```hcl
# ❌ Bad — region-specific and becomes outdated
ami = "ami-0c02fb55956c7d316"

# ✅ Good — always resolves the latest AMI dynamically
ami = local.AMI_ID
```

### 3. Use `locals.tf` to Centralize Derived Values
Placing the AMI data source and local in a dedicated `locals.tf` separates infrastructure data concerns from resource logic in `main.tf`. This improves maintainability.

### 4. Use `count.index + 1` for Human-Readable Naming
Zero-indexed naming (`instance-0`) is confusing. Always add `+1` to start human-facing names from `1`.

### 5. Use `for` Expressions in Outputs for `count`-Based Resources
When a resource uses `count`, it becomes a list. Use `for` expressions to extract specific attributes:
```hcl
value = [for inst in aws_instance.xfusion_ec2 : inst.tags.Name]
```

### 6. Scale Up by Changing a Single Variable
The power of `count` is that adding or removing instances requires only one change in `terraform.tfvars`:
```hcl
KKE_INSTANCE_COUNT = 5   # Scale from 3 to 5 with one line change
```

### 7. Always Lock Provider Versions
The `data "aws_ami"` source behavior depends on the provider version. Commit `.terraform.lock.hcl` to version control to ensure consistent AMI resolution across team members and CI/CD runs.

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Count | Configuration | Names |
|----------|-------|---------------|-------|
| `aws_instance.xfusion_ec2` | 3 | `t2.micro`, `xfusion-key`, Latest Amazon Linux 2 | `xfusion-instance-1/2/3` |

### Files Created

| File | Purpose |
|------|---------|
| `main.tf` | EC2 resource with `count = 3` and dynamic naming |
| `variables.tf` | 4 variable declarations (count, type, key, prefix) |
| `terraform.tfvars` | Assigns all 4 variable values |
| `locals.tf` | Data source + `AMI_ID` local for dynamic AMI resolution |
| `outputs.tf` | `kke_instance_names` — list of all instance names |

### Task Completion Checklist

- [x] ✅ 3 EC2 instances created using the `count` parameter
- [x] ✅ Instances named `xfusion-instance-1`, `xfusion-instance-2`, `xfusion-instance-3`
- [x] ✅ Instance type is `t2.micro`
- [x] ✅ Key pair `xfusion-key` attached to all instances
- [x] ✅ All resources defined in a **single `main.tf`** file
- [x] ✅ `variables.tf` contains `KKE_INSTANCE_COUNT`, `KKE_INSTANCE_TYPE`, `KKE_KEY_NAME`, `KKE_INSTANCE_PREFIX`
- [x] ✅ `locals.tf` defines `AMI_ID` using `data "aws_ami"` for latest Amazon Linux 2
- [x] ✅ `terraform.tfvars` assigns all variable values
- [x] ✅ `outputs.tf` exports `kke_instance_names` as a list
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply -auto-approve` — **3 resources added**
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task demonstrates Terraform's **`count` meta-argument** — the primary mechanism for provisioning fleets of identical resources from a single resource block. Combined with a **`data "aws_ami"` source** for dynamic image resolution and a **`for` expression in outputs**, this pattern is a foundation of scalable, DRY (Don't Repeat Yourself) Infrastructure-as-Code. Changing `KKE_INSTANCE_COUNT` in `terraform.tfvars` is all it takes to scale this fleet up or down.
