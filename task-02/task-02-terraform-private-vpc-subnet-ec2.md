# 🌟 Task 02 - Provision a Private AWS VPC, Subnet, and EC2 Instance Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is expanding their AWS infrastructure and requires the setup of a **private Virtual Private Cloud (VPC)** along with a **private subnet** and a secure **EC2 instance**. This configuration ensures that resources deployed within the environment remain isolated from external networks and can only communicate within the VPC, following best practices for private AWS workloads.

**Requirements:**
- Create a **VPC** named **`nautilus-priv-vpc`** with CIDR block `10.0.0.0/16`
- Create a **Subnet** named **`nautilus-priv-subnet`** inside the VPC with CIDR block `10.0.1.0/24` — **auto-assign public IP must be disabled**
- Create an **EC2 instance** named **`nautilus-priv-ec2`** inside the subnet with instance type **`t2.micro`**
- Ensure the EC2 instance's **Security Group** allows access **only from within the VPC's CIDR block**
- All resources must be defined in a single **`main.tf`** file (no additional `.tf` resource files)
- Use a **`variables.tf`** file with:
  - `KKE_VPC_CIDR` for the VPC CIDR block
  - `KKE_SUBNET_CIDR` for the Subnet CIDR block
- Use an **`outputs.tf`** file with:
  - `KKE_vpc_name` for the VPC name
  - `KKE_subnet_name` for the Subnet name
  - `KKE_ec2_private` for the EC2 instance name
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Provision a fully private AWS network environment — VPC, subnet, security group, and EC2 instance — using Terraform with proper variable management and output configuration.

💡 **Note:** Disabling `map_public_ip_on_launch` and restricting security group ingress to the VPC CIDR block are critical security measures that ensure the EC2 instance remains inaccessible from the public internet.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- AWS VPC (Virtual Private Cloud) with `/16` CIDR block
- AWS Subnet (private) within VPC with `/24` CIDR block, public IP auto-assign disabled
- AWS Security Group restricting ingress to VPC CIDR only
- AWS EC2 Instance (`t2.micro`) deployed inside the private subnet

**Working Directory:** `/home/bob/terraform`

**Network Configuration:**
| Resource | Name | CIDR / Type |
|----------|------|-------------|
| VPC | `nautilus-priv-vpc` | `10.0.0.0/16` (65,536 IPs) |
| Subnet | `nautilus-priv-subnet` | `10.0.1.0/24` (256 IPs, private) |
| Security Group | `nautilus-priv-sg` | Ingress: VPC CIDR only |
| EC2 Instance | `nautilus-priv-ec2` | `t2.micro`, Ubuntu AMI |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **VPC Resource (`aws_vpc`):** Isolated virtual network with a `/16` IP space
- **Subnet Resource (`aws_subnet`):** Private subdivision within the VPC — no public IP assignment
- **Security Group (`aws_security_group`):** Network firewall restricting all inbound traffic to the VPC CIDR block only
- **EC2 Instance (`aws_instance`):** `t2.micro` compute instance launched in the private subnet, protected by the security group
- **Variable Management:** Centralized CIDR definitions using `variables.tf` and `terraform.tfvars`
- **Output Configuration:** Exports resource names for verification and downstream reference

### 🎯 Implementation Strategy
1. Define CIDR variables in `variables.tf`
2. Assign values in `terraform.tfvars`
3. Create VPC, subnet, security group, and EC2 instance in `main.tf`
4. Configure output values in `outputs.tf`
5. Initialize, validate, format, plan, apply, and verify the infrastructure

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf           # VPC, Subnet, Security Group, EC2 Instance definitions
├── variables.tf      # Variable declarations (CIDR blocks)
├── terraform.tfvars  # Variable value assignments
└── outputs.tf        # Output value definitions
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

**Purpose:** Ensure we're working in the correct Terraform directory as specified in the requirements.

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

Create `variables.tf` to declare the input variables:

```bash
cat > variables.tf << 'EOF'
variable "KKE_VPC_CIDR" {
  type        = string
  description = "CIDR block for the private VPC"
}

variable "KKE_SUBNET_CIDR" {
  type        = string
  description = "CIDR block for the private subnet"
}
EOF
```

**Purpose:** Define the input variables that drive CIDR block values throughout the configuration.

**Configuration Breakdown:**
- `variable "KKE_VPC_CIDR"` — Declares the variable for the VPC CIDR block
- `variable "KKE_SUBNET_CIDR"` — Declares the variable for the subnet CIDR block
- `type = string` — Enforces string type for both variables
- `description` — Documents the intended purpose of each variable
- **Best Practice:** Separating variable declarations from values enables environment-specific overrides without modifying resource code

**File Content (`variables.tf`):**
```hcl
variable "KKE_VPC_CIDR" {
  type        = string
  description = "CIDR block for the private VPC"
}

variable "KKE_SUBNET_CIDR" {
  type        = string
  description = "CIDR block for the private subnet"
}
```

---

### Step 3: Create Variable Values File

Create `terraform.tfvars` to assign concrete values to the declared variables:

```bash
cat > terraform.tfvars << 'EOF'
KKE_VPC_CIDR    = "10.0.0.0/16"
KKE_SUBNET_CIDR = "10.0.1.0/24"
EOF
```

**Purpose:** Provide actual CIDR values for each variable, separating data from infrastructure logic.

**Configuration Breakdown:**
- `KKE_VPC_CIDR = "10.0.0.0/16"` — VPC address space covering 65,536 IPs
- `KKE_SUBNET_CIDR = "10.0.1.0/24"` — Subnet carved from VPC space, covering 256 IPs
- **Best Practice:** Using `.tfvars` files allows you to swap values per environment (dev/staging/prod) without touching resource definitions

**File Content (`terraform.tfvars`):**
```hcl
KKE_VPC_CIDR    = "10.0.0.0/16"
KKE_SUBNET_CIDR = "10.0.1.0/24"
```

---

### Step 4: Create Main Configuration File

Create `main.tf` with VPC, Subnet, Security Group, and EC2 Instance resources:

```bash
cat > main.tf << 'EOF'
# --------------------------
# VPC
# --------------------------
resource "aws_vpc" "nautilus_priv_vpc" {
  cidr_block = var.KKE_VPC_CIDR

  tags = {
    Name = "nautilus-priv-vpc"
  }
}

# --------------------------
# SUBNET (private)
# --------------------------
resource "aws_subnet" "nautilus_priv_subnet" {
  vpc_id                  = aws_vpc.nautilus_priv_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false   # auto-assign public IP disabled

  tags = {
    Name = "nautilus-priv-subnet"
  }
}

# --------------------------
# SECURITY GROUP
# --------------------------
resource "aws_security_group" "nautilus_priv_sg" {
  name        = "nautilus-priv-sg"
  description = "Allow traffic only from VPC CIDR"
  vpc_id      = aws_vpc.nautilus_priv_vpc.id

  ingress {
    description = "Allow all inbound traffic only from VPC CIDR"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "nautilus-priv-sg"
  }
}

# --------------------------
# EC2 INSTANCE
# --------------------------
resource "aws_instance" "nautilus_priv_ec2" {
  ami           = "ami-0c02fb55956c7d316"  # Ubuntu AMI (us-east-1)
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.nautilus_priv_subnet.id
  vpc_security_group_ids = [
    aws_security_group.nautilus_priv_sg.id
  ]

  tags = {
    Name = "nautilus-priv-ec2"
  }
}
EOF
```

**Purpose:** Define all four AWS resources forming the private infrastructure — VPC, subnet, security group, and EC2 instance — in a single file as required.

**Configuration Breakdown:**

**VPC Resource (`aws_vpc.nautilus_priv_vpc`):**
- `cidr_block = var.KKE_VPC_CIDR` — Uses the variable to define the IP address space
- `tags.Name = "nautilus-priv-vpc"` — Labels the resource as required

**Subnet Resource (`aws_subnet.nautilus_priv_subnet`):**
- `vpc_id = aws_vpc.nautilus_priv_vpc.id` — Associates the subnet with the VPC (implicit dependency)
- `cidr_block = var.KKE_SUBNET_CIDR` — Subnet IP range carved from the VPC space
- `map_public_ip_on_launch = false` — **Critical:** Prevents auto-assignment of public IPs, keeping instances private
- `tags.Name = "nautilus-priv-subnet"` — Labels the resource as required

**Security Group Resource (`aws_security_group.nautilus_priv_sg`):**
- `vpc_id` — Attaches the security group to the correct VPC
- `ingress` block — Allows **all protocols** (`protocol = "-1"`) but **only from within the VPC CIDR** (`cidr_blocks = [var.KKE_VPC_CIDR]`), ensuring no external access
- `egress` block — Allows all outbound traffic (standard for private instances to pull updates)

**EC2 Instance Resource (`aws_instance.nautilus_priv_ec2`):**
- `ami` — Ubuntu AMI for us-east-1 region
- `instance_type = "t2.micro"` — Cost-effective instance type as required
- `subnet_id` — Deploys instance into the private subnet
- `vpc_security_group_ids` — Attaches the security group to enforce VPC-only access

**Resource Dependency Chain:**
| Creation Order | Resource | Depends On |
|---|---|---|
| 1st | `aws_vpc.nautilus_priv_vpc` | None |
| 2nd | `aws_subnet.nautilus_priv_subnet` | VPC ID |
| 2nd | `aws_security_group.nautilus_priv_sg` | VPC ID |
| 3rd | `aws_instance.nautilus_priv_ec2` | Subnet ID + Security Group ID |

**File Content (`main.tf`):**
```hcl
# --------------------------
# VPC
# --------------------------
resource "aws_vpc" "nautilus_priv_vpc" {
  cidr_block = var.KKE_VPC_CIDR

  tags = {
    Name = "nautilus-priv-vpc"
  }
}

# --------------------------
# SUBNET (private)
# --------------------------
resource "aws_subnet" "nautilus_priv_subnet" {
  vpc_id                  = aws_vpc.nautilus_priv_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false

  tags = {
    Name = "nautilus-priv-subnet"
  }
}

# --------------------------
# SECURITY GROUP
# --------------------------
resource "aws_security_group" "nautilus_priv_sg" {
  name        = "nautilus-priv-sg"
  description = "Allow traffic only from VPC CIDR"
  vpc_id      = aws_vpc.nautilus_priv_vpc.id

  ingress {
    description = "Allow all inbound traffic only from VPC CIDR"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "nautilus-priv-sg"
  }
}

# --------------------------
# EC2 INSTANCE
# --------------------------
resource "aws_instance" "nautilus_priv_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.nautilus_priv_subnet.id
  vpc_security_group_ids = [
    aws_security_group.nautilus_priv_sg.id
  ]

  tags = {
    Name = "nautilus-priv-ec2"
  }
}
```

---

### Step 5: Create Outputs Configuration File

Create `outputs.tf` to export resource names for verification:

```bash
cat > outputs.tf << 'EOF'
output "KKE_vpc_name" {
  value = aws_vpc.nautilus_priv_vpc.tags["Name"]
}

output "KKE_subnet_name" {
  value = aws_subnet.nautilus_priv_subnet.tags["Name"]
}

output "KKE_ec2_private" {
  value = aws_instance.nautilus_priv_ec2.tags["Name"]
}
EOF
```

**Purpose:** Expose key resource identifiers after `terraform apply` for verification and downstream reference.

**Configuration Breakdown:**
- `output "KKE_vpc_name"` — Exports the VPC name from its tag
- `output "KKE_subnet_name"` — Exports the subnet name from its tag
- `output "KKE_ec2_private"` — Exports the EC2 instance name from its tag
- `value = resource.type.name.tags["Name"]` — Direct tag extraction pattern for name outputs

**File Content (`outputs.tf`):**
```hcl
output "KKE_vpc_name" {
  value = aws_vpc.nautilus_priv_vpc.tags["Name"]
}

output "KKE_subnet_name" {
  value = aws_subnet.nautilus_priv_subnet.tags["Name"]
}

output "KKE_ec2_private" {
  value = aws_instance.nautilus_priv_ec2.tags["Name"]
}
```

---

### Step 6: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Purpose:** Confirm all required files are created before initializing Terraform.

**Expected Output:**
```
total 16
drwxr-xr-x 2 bob bob 4096 Feb 25 10:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 09:55 ..
-rw-r--r-- 1 bob bob 1024 Feb 25 10:00 main.tf
-rw-r--r-- 1 bob bob  345 Feb 25 10:00 outputs.tf
-rw-r--r-- 1 bob bob   72 Feb 25 10:00 terraform.tfvars
-rw-r--r-- 1 bob bob  210 Feb 25 10:00 variables.tf
```

**Success Indicators:**
- ✅ All four files present: `main.tf`, `variables.tf`, `terraform.tfvars`, `outputs.tf`
- ✅ Files have appropriate read permissions
- ✅ Files are in the correct working directory

---

### Step 7: Initialize Terraform

```bash
terraform init
```

**Purpose:** Initialize the Terraform working directory, download the AWS provider plugin, and prepare the backend.

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

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

**Success Indicators:**
- ✅ Backend initialized successfully
- ✅ AWS provider downloaded and installed
- ✅ `.terraform/` directory created with provider binaries
- ✅ `.terraform.lock.hcl` generated to lock provider versions

**Created Files/Directories:**
```bash
/home/bob/terraform/
├── .terraform/           # Provider plugins directory
├── .terraform.lock.hcl   # Provider version lock file
├── main.tf
├── outputs.tf
├── terraform.tfvars
└── variables.tf
```

---

### Step 8: Validate Configuration

```bash
terraform validate
```

**Purpose:** Verify the syntax and internal consistency of all Terraform configuration files before planning.

**Expected Output:**
```
Success! The configuration is valid.
```

**What's Validated:**
- ✅ HCL syntax correctness
- ✅ Variable references are resolvable
- ✅ Resource attribute names are correct for the AWS provider
- ✅ Security group ingress/egress rule structures are valid
- ✅ EC2 instance references properly link to subnet and security group

---

### Step 9: Format Configuration Files

```bash
terraform fmt
```

**Purpose:** Automatically format all Terraform files to the canonical HCL style.

**Expected Output:**
```
main.tf
outputs.tf
terraform.tfvars
variables.tf
```

**Formatting Applied:**
- ✅ Consistent 2-space indentation
- ✅ Proper alignment of `=` signs in blocks
- ✅ Standardized blank line usage between blocks
- ✅ Canonical code style throughout all files

---

### Step 10: Review Execution Plan

```bash
terraform plan
```

**Purpose:** Preview all changes Terraform will make to AWS infrastructure before applying.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.nautilus_priv_ec2 will be created
  + resource "aws_instance" "nautilus_priv_ec2" {
      + ami                                  = "ami-0c02fb55956c7d316"
      + arn                                  = (known after apply)
      + id                                   = (known after apply)
      + instance_type                        = "t2.micro"
      + private_ip                           = (known after apply)
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "nautilus-priv-ec2"
        }
      + vpc_security_group_ids               = (known after apply)
    }

  # aws_security_group.nautilus_priv_sg will be created
  + resource "aws_security_group" "nautilus_priv_sg" {
      + arn                    = (known after apply)
      + description            = "Allow traffic only from VPC CIDR"
      + egress                 = [
          + {
              + cidr_blocks      = ["0.0.0.0/0"]
              + from_port        = 0
              + protocol         = "-1"
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = ["10.0.0.0/16"]
              + from_port        = 0
              + protocol         = "-1"
              + to_port          = 0
            },
        ]
      + name                   = "nautilus-priv-sg"
      + tags                   = {
          + "Name" = "nautilus-priv-sg"
        }
      + vpc_id                 = (known after apply)
    }

  # aws_subnet.nautilus_priv_subnet will be created
  + resource "aws_subnet" "nautilus_priv_subnet" {
      + arn                                            = (known after apply)
      + cidr_block                                     = "10.0.1.0/24"
      + id                                             = (known after apply)
      + map_public_ip_on_launch                        = false
      + tags                                           = {
          + "Name" = "nautilus-priv-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # aws_vpc.nautilus_priv_vpc will be created
  + resource "aws_vpc" "nautilus_priv_vpc" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + enable_dns_support                   = true
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + tags                                 = {
          + "Name" = "nautilus-priv-vpc"
        }
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + KKE_ec2_private = "nautilus-priv-ec2"
  + KKE_subnet_name = "nautilus-priv-subnet"
  + KKE_vpc_name    = "nautilus-priv-vpc"

───────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
```

**Plan Summary:**
- **Resources to Create:** 4 (VPC + Subnet + Security Group + EC2 Instance)
- **Resources to Change:** 0
- **Resources to Destroy:** 0
- **Outputs to Create:** 3 (VPC name, Subnet name, EC2 name)

**Success Indicators:**
- ✅ All 4 resources planned for creation
- ✅ `map_public_ip_on_launch = false` confirmed on subnet
- ✅ Security group ingress restricted to `10.0.0.0/16`
- ✅ EC2 instance uses `t2.micro` and correct subnet
- ✅ All 3 output values configured correctly

---

### Step 11: Apply Configuration

```bash
terraform apply -auto-approve
```

**Purpose:** Execute the planned changes and provision all four resources in AWS.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.nautilus_priv_ec2 will be created
  ...
  # aws_security_group.nautilus_priv_sg will be created
  ...
  # aws_subnet.nautilus_priv_subnet will be created
  ...
  # aws_vpc.nautilus_priv_vpc will be created
  ...

Plan: 4 to add, 0 to change, 0 to destroy.

aws_vpc.nautilus_priv_vpc: Creating...
aws_vpc.nautilus_priv_vpc: Creation complete after 2s [id=vpc-0a1b2c3d4e5f6789a]
aws_subnet.nautilus_priv_subnet: Creating...
aws_security_group.nautilus_priv_sg: Creating...
aws_subnet.nautilus_priv_subnet: Creation complete after 1s [id=subnet-0b1c2d3e4f5a6789b]
aws_security_group.nautilus_priv_sg: Creation complete after 2s [id=sg-0c2d3e4f5a6789c]
aws_instance.nautilus_priv_ec2: Creating...
aws_instance.nautilus_priv_ec2: Still creating... [10s elapsed]
aws_instance.nautilus_priv_ec2: Creation complete after 32s [id=i-0d3e4f5a6b789d1e2]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

KKE_ec2_private = "nautilus-priv-ec2"
KKE_subnet_name = "nautilus-priv-subnet"
KKE_vpc_name    = "nautilus-priv-vpc"
```

**Success Indicators:**
- ✅ VPC created first (prerequisite for subnet and security group)
- ✅ Subnet and Security Group created in parallel after VPC
- ✅ EC2 instance created last (depends on both subnet and security group)
- ✅ All four resources have unique AWS-assigned IDs
- ✅ All three output values displayed correctly
- ✅ No errors during resource creation

**Creation Order Verification:**
| Order | Resource | Reason |
|-------|----------|--------|
| 1st | `aws_vpc.nautilus_priv_vpc` | No dependencies |
| 2nd (parallel) | `aws_subnet.nautilus_priv_subnet` | Needs VPC ID |
| 2nd (parallel) | `aws_security_group.nautilus_priv_sg` | Needs VPC ID |
| 3rd | `aws_instance.nautilus_priv_ec2` | Needs Subnet ID + SG ID |

---

### Step 12: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Purpose:** Confirm the deployed infrastructure matches the Terraform configuration with zero pending changes.

**Expected Output:**
```
aws_vpc.nautilus_priv_vpc: Refreshing state... [id=vpc-0a1b2c3d4e5f6789a]
aws_subnet.nautilus_priv_subnet: Refreshing state... [id=subnet-0b1c2d3e4f5a6789b]
aws_security_group.nautilus_priv_sg: Refreshing state... [id=sg-0c2d3e4f5a6789c]
aws_instance.nautilus_priv_ec2: Refreshing state... [id=i-0d3e4f5a6b789d1e2]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ All 4 resources refreshed successfully from AWS
- ✅ No drift detected between remote state and actual AWS resources
- ✅ Configuration is complete, accurate, and task-ready for submission

**⚠️ Important:** This step is **required before task submission**. The "No changes" message is the definitive confirmation of successful infrastructure provisioning.

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Purpose:** List all resources tracked in the current Terraform state file.

**Expected Output:**
```
aws_instance.nautilus_priv_ec2
aws_security_group.nautilus_priv_sg
aws_subnet.nautilus_priv_subnet
aws_vpc.nautilus_priv_vpc
```

**Success Indicators:**
- ✅ All 4 resources present in state
- ✅ Resource names match configuration exactly

---

### Step 2: Check VPC Details

```bash
terraform state show aws_vpc.nautilus_priv_vpc
```

**Purpose:** Display detailed information about the provisioned VPC.

**Expected Output:**
```
# aws_vpc.nautilus_priv_vpc:
resource "aws_vpc" "nautilus_priv_vpc" {
    arn                              = "arn:aws:ec2:us-east-1:123456789012:vpc/vpc-0a1b2c3d4e5f6789a"
    cidr_block                       = "10.0.0.0/16"
    enable_dns_hostnames             = false
    enable_dns_support               = true
    id                               = "vpc-0a1b2c3d4e5f6789a"
    instance_tenancy                 = "default"
    tags                             = {
        "Name" = "nautilus-priv-vpc"
    }
}
```

**Verification Points:**
- ✅ CIDR block is `10.0.0.0/16`
- ✅ Name tag is `nautilus-priv-vpc`
- ✅ Resource has a valid VPC ID and ARN

---

### Step 3: Check Subnet Details

```bash
terraform state show aws_subnet.nautilus_priv_subnet
```

**Purpose:** Confirm the subnet is private and correctly configured.

**Expected Output:**
```
# aws_subnet.nautilus_priv_subnet:
resource "aws_subnet" "nautilus_priv_subnet" {
    arn                                            = "arn:aws:ec2:us-east-1:123456789012:subnet/subnet-0b1c2d3e4f5a6789b"
    cidr_block                                     = "10.0.1.0/24"
    id                                             = "subnet-0b1c2d3e4f5a6789b"
    map_public_ip_on_launch                        = false
    tags                                           = {
        "Name" = "nautilus-priv-subnet"
    }
    vpc_id                                         = "vpc-0a1b2c3d4e5f6789a"
}
```

**Verification Points:**
- ✅ CIDR block is `10.0.1.0/24` (within VPC range)
- ✅ `map_public_ip_on_launch = false` — auto-assign IP is disabled
- ✅ `vpc_id` matches the created VPC
- ✅ Name tag is `nautilus-priv-subnet`

---

### Step 4: Check Security Group Details

```bash
terraform state show aws_security_group.nautilus_priv_sg
```

**Purpose:** Confirm that the security group restricts ingress to the VPC CIDR only.

**Expected Output:**
```
# aws_security_group.nautilus_priv_sg:
resource "aws_security_group" "nautilus_priv_sg" {
    description = "Allow traffic only from VPC CIDR"
    egress      = [
        {
            cidr_blocks = ["0.0.0.0/0"]
            from_port   = 0
            protocol    = "-1"
            to_port     = 0
        },
    ]
    id          = "sg-0c2d3e4f5a6789c"
    ingress     = [
        {
            cidr_blocks = ["10.0.0.0/16"]
            from_port   = 0
            protocol    = "-1"
            to_port     = 0
        },
    ]
    name        = "nautilus-priv-sg"
    tags        = {
        "Name" = "nautilus-priv-sg"
    }
    vpc_id      = "vpc-0a1b2c3d4e5f6789a"
}
```

**Verification Points:**
- ✅ Ingress `cidr_blocks` is `["10.0.0.0/16"]` — VPC CIDR only
- ✅ No `0.0.0.0/0` in ingress — no public internet access
- ✅ Egress allows all outbound traffic
- ✅ Attached to the correct VPC

---

### Step 5: Check EC2 Instance Details

```bash
terraform state show aws_instance.nautilus_priv_ec2
```

**Purpose:** Confirm the EC2 instance is correctly placed in the private subnet with the right specs.

**Expected Output:**
```
# aws_instance.nautilus_priv_ec2:
resource "aws_instance" "nautilus_priv_ec2" {
    ami                    = "ami-0c02fb55956c7d316"
    id                     = "i-0d3e4f5a6b789d1e2"
    instance_type          = "t2.micro"
    private_ip             = "10.0.1.x"
    subnet_id              = "subnet-0b1c2d3e4f5a6789b"
    tags                   = {
        "Name" = "nautilus-priv-ec2"
    }
    vpc_security_group_ids = ["sg-0c2d3e4f5a6789c"]
}
```

**Verification Points:**
- ✅ Instance type is `t2.micro`
- ✅ Placed in the correct private subnet
- ✅ `private_ip` falls within the subnet CIDR `10.0.1.0/24`
- ✅ Security group correctly attached
- ✅ No public IP assigned (private subnet)
- ✅ Name tag is `nautilus-priv-ec2`

---

### Step 6: View Output Values

```bash
terraform output
```

**Purpose:** Display all configured output values.

**Expected Output:**
```
KKE_ec2_private = "nautilus-priv-ec2"
KKE_subnet_name = "nautilus-priv-subnet"
KKE_vpc_name    = "nautilus-priv-vpc"
```

**Individual Output Access:**
```bash
terraform output KKE_vpc_name
terraform output KKE_subnet_name
terraform output KKE_ec2_private
```

**Expected Output:**
```
"nautilus-priv-vpc"
"nautilus-priv-subnet"
"nautilus-priv-ec2"
```

---

### Step 7: AWS CLI Verification (Optional)

```bash
# Verify VPC exists in AWS
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=nautilus-priv-vpc"

# Verify Subnet exists in AWS
aws ec2 describe-subnets --filters "Name=tag:Name,Values=nautilus-priv-subnet"

# Verify EC2 instance exists in AWS
aws ec2 describe-instances --filters "Name=tag:Name,Values=nautilus-priv-ec2"

# Verify Security Group in AWS
aws ec2 describe-security-groups --filters "Name=tag:Name,Values=nautilus-priv-sg"
```

**Purpose:** Confirm resources exist in AWS and match the Terraform state independently.

---

## 🔍 Code Analysis

### Complete Solution Structure

**File Organization:**
```bash
/home/bob/terraform/
├── main.tf              # All resource definitions (VPC, Subnet, SG, EC2)
├── variables.tf         # Variable declarations (CIDR blocks)
├── terraform.tfvars     # Variable value assignments
└── outputs.tf           # Output value definitions (names of all 3 resources)
```

### Security Architecture Analysis

**Why restrict ingress to VPC CIDR only?**

```hcl
# CORRECT — Private access only (VPC-internal traffic)
ingress {
  from_port   = 0
  to_port     = 0
  protocol    = "-1"
  cidr_blocks = [var.KKE_VPC_CIDR]   # 10.0.0.0/16 only
}

# ❌ WRONG for private instances — would expose to internet
ingress {
  from_port   = 0
  to_port     = 0
  protocol    = "-1"
  cidr_blocks = ["0.0.0.0/0"]        # Any IP — never do this for private resources!
}
```

By setting `cidr_blocks = [var.KKE_VPC_CIDR]`, the security group ensures that **only resources within the same VPC** (10.0.0.0/16) can establish inbound connections to the EC2 instance.

### Variable Flow Diagram

```
variables.tf (Declaration)
    ↓
terraform.tfvars (Value Assignment)
    ↓
main.tf (Variable Usage: var.KKE_VPC_CIDR, var.KKE_SUBNET_CIDR)
    ↓
AWS Resources (CIDR applied to VPC, Subnet, Security Group)
    ↓
outputs.tf (Tag value extraction)
    ↓
Terminal Output (KKE_vpc_name, KKE_subnet_name, KKE_ec2_private)
```

### CIDR Block Relationship

```
VPC:    10.0.0.0/16    (10.0.0.0 - 10.0.255.255)    65,536 IPs
         └── Subnet: 10.0.1.0/24  (10.0.1.0 - 10.0.1.255)    256 IPs
                          └── EC2: private_ip → 10.0.1.x
```

### Private vs Public Subnet Key Differences

| Feature | Private Subnet | Public Subnet |
|---------|---------------|---------------|
| `map_public_ip_on_launch` | `false` ✅ | `true` |
| Internet Gateway Route | Not needed | Required |
| Public IP on EC2 | None | Assigned |
| External accessibility | VPC-internal only | Via public IP |
| Use Case | Databases, internal apps | Web servers, load balancers |

### Implicit Dependency Chain

Terraform automatically detects these dependencies from resource attribute references:

```
aws_vpc.nautilus_priv_vpc
    ├── aws_subnet.nautilus_priv_subnet        (via vpc_id)
    └── aws_security_group.nautilus_priv_sg    (via vpc_id)
            └── aws_instance.nautilus_priv_ec2 (via subnet_id + vpc_security_group_ids)
```

No `depends_on` is needed here because **all dependencies are implicit** through resource attribute references — Terraform's graph engine resolves the creation order automatically.

---

## 🧪 Testing & Validation

### Pre-Apply Validation Checklist

```bash
# 1. Validate syntax
terraform validate

# 2. Preview changes
terraform plan

# 3. Confirm plan shows 4 resources to create
# Plan: 4 to add, 0 to change, 0 to destroy.
```

### Post-Apply Verification Checklist

```bash
# 1. Verify all 4 resources in state
terraform state list

# 2. Confirm VPC configuration
terraform state show aws_vpc.nautilus_priv_vpc

# 3. Confirm subnet is private (map_public_ip_on_launch = false)
terraform state show aws_subnet.nautilus_priv_subnet

# 4. Confirm SG ingress restricted to VPC CIDR
terraform state show aws_security_group.nautilus_priv_sg

# 5. Confirm EC2 instance specs
terraform state show aws_instance.nautilus_priv_ec2

# 6. View all output values
terraform output

# 7. Final verification — must show "No changes"
terraform plan
```

### Key Validation Points

| Check | Expected Value | Command |
|-------|---------------|---------|
| VPC CIDR | `10.0.0.0/16` | `terraform state show aws_vpc.nautilus_priv_vpc` |
| Subnet CIDR | `10.0.1.0/24` | `terraform state show aws_subnet.nautilus_priv_subnet` |
| Auto-assign IP | `false` | `terraform state show aws_subnet.nautilus_priv_subnet` |
| SG Ingress CIDR | `10.0.0.0/16` | `terraform state show aws_security_group.nautilus_priv_sg` |
| EC2 Type | `t2.micro` | `terraform state show aws_instance.nautilus_priv_ec2` |
| Final Plan | "No changes" | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: AMI Not Found

**Error:**
```
Error: reading EC2 AMI (ami-0c02fb55956c7d316): InvalidAMIID.NotFound
```

**Cause:** The AMI ID is region-specific. If the AWS provider is configured to a different region, the AMI will not exist.

**Solution:**
```bash
# Find a valid Ubuntu AMI for your region
aws ec2 describe-images \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*" \
            "Name=state,Values=available" \
  --query "Images | sort_by(@, &CreationDate) | [-1].ImageId" \
  --output text

# Update main.tf with the correct AMI ID for your region
```

---

### Issue 2: Security Group Ingress Too Broad

**Error (Logical):**
```
The task requires access only from within the VPC — not 0.0.0.0/0
```

**Solution:** Ensure `cidr_blocks` in the `ingress` block uses the VPC CIDR variable, not open internet:
```hcl
# Correct
cidr_blocks = [var.KKE_VPC_CIDR]   # "10.0.0.0/16"

# Wrong
cidr_blocks = ["0.0.0.0/0"]
```

---

### Issue 3: Public IP Auto-Assign Not Disabled

**Error (Logical):**
```
Task validation fails — subnet has auto-assign public IP enabled
```

**Solution:** Confirm `map_public_ip_on_launch = false` is explicitly set in the subnet resource:
```hcl
resource "aws_subnet" "nautilus_priv_subnet" {
  map_public_ip_on_launch = false   # Must be explicitly false
  # ...
}
```

---

### Issue 4: Variable Values Not Loaded

**Error:**
```
var.KKE_VPC_CIDR is not set
```

**Cause:** `terraform.tfvars` file may be missing or misnamed.

**Solution:**
```bash
# Verify the tfvars file exists and has correct content
cat terraform.tfvars

# If running plan/apply manually with a different filename
terraform plan -var-file="custom.tfvars"
```

---

### Issue 5: EC2 Instance Taking Too Long

**Symptom:** `aws_instance.nautilus_priv_ec2: Still creating...` hangs for several minutes.

**Cause:** EC2 instance provisioning typically takes 30–60 seconds. This is normal.

**Solution:** Wait for the instance to reach the `running` state. If it times out, check the AWS Console for errors related to subnet availability or IAM permissions.

---

### Issue 6: Terraform Plan Shows Unwanted Changes

**Error:**
```
Plan: 0 to add, 1 to change, 0 to destroy.
```

**Cause:** A resource attribute in `main.tf` doesn't match the actual AWS resource state.

**Solution:**
```bash
# Refresh the state to sync with actual AWS resources
terraform refresh

# Re-run plan to confirm current differences
terraform plan

# If changes are legitimate drift corrections, re-apply
terraform apply -auto-approve
```

---

## 📚 Best Practices

### 1. Use Variables for CIDR Blocks
Hardcoding CIDR values makes configurations rigid and error-prone. Using variables (`var.KKE_VPC_CIDR`) centralizes values and makes changes safer.

### 2. Always Disable Public IP on Private Subnets
```hcl
map_public_ip_on_launch = false
```
This is a security-critical setting. Private instances should never receive public IPs unless explicitly intended.

### 3. Principle of Least Privilege for Security Groups
Restrict ingress to only what is necessary:
```hcl
# ✅ Good — CIDR-restricted access
cidr_blocks = [var.KKE_VPC_CIDR]

# ❌ Bad — open to the world
cidr_blocks = ["0.0.0.0/0"]
```

### 4. Use Tags Consistently
Tags are essential for cost allocation, resource tracking, and automation. Always apply `Name` tags and consider adding `Environment`, `Owner`, and `Project` tags in production.

### 5. Consolidate Resources in a Single `main.tf` (When Required)
For small, focused infrastructure provisioning tasks like this one, keeping all resources in `main.tf` is clear and maintainable. For larger environments, split resources into logical files (e.g., `vpc.tf`, `ec2.tf`).

### 6. Always Verify with `terraform plan` Post-Apply
Running `terraform plan` after `terraform apply` confirms your infrastructure exactly matches the declared configuration — no drift, no missing resources.

### 7. Lock Provider Versions
The `.terraform.lock.hcl` file (auto-generated by `terraform init`) locks provider versions. Commit this file to version control to ensure consistent behavior across team members and CI/CD pipelines.

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Name | Configuration |
|----------|------|---------------|
| **VPC** | `nautilus-priv-vpc` | CIDR `10.0.0.0/16` |
| **Subnet** | `nautilus-priv-subnet` | CIDR `10.0.1.0/24`, auto-IP disabled |
| **Security Group** | `nautilus-priv-sg` | Ingress: VPC CIDR only |
| **EC2 Instance** | `nautilus-priv-ec2` | `t2.micro`, private subnet |

### Files Created

| File | Purpose |
|------|---------|
| `main.tf` | All resource definitions (VPC, Subnet, SG, EC2) |
| `variables.tf` | Variable declarations (`KKE_VPC_CIDR`, `KKE_SUBNET_CIDR`) |
| `terraform.tfvars` | CIDR value assignments |
| `outputs.tf` | Exports `KKE_vpc_name`, `KKE_subnet_name`, `KKE_ec2_private` |

### Task Completion Checklist

- [x] ✅ VPC `nautilus-priv-vpc` created with CIDR `10.0.0.0/16`
- [x] ✅ Subnet `nautilus-priv-subnet` created with CIDR `10.0.1.0/24`
- [x] ✅ Subnet auto-assign public IP is **disabled** (`map_public_ip_on_launch = false`)
- [x] ✅ Security group `nautilus-priv-sg` created — ingress **restricted to VPC CIDR**
- [x] ✅ EC2 instance `nautilus-priv-ec2` created with type `t2.micro`
- [x] ✅ EC2 instance placed in private subnet with correct security group
- [x] ✅ All resources defined in a **single `main.tf`** file
- [x] ✅ `variables.tf` contains `KKE_VPC_CIDR` and `KKE_SUBNET_CIDR`
- [x] ✅ `terraform.tfvars` assigns correct CIDR values
- [x] ✅ `outputs.tf` exports `KKE_vpc_name`, `KKE_subnet_name`, `KKE_ec2_private`
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply -auto-approve` — **4 resources added**
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task demonstrates a complete **private AWS network stack** provisioned with Terraform. The combination of `map_public_ip_on_launch = false` on the subnet and a security group ingress restricted to the VPC CIDR ensures the EC2 instance is fully isolated from the public internet — accessible **only from resources within the same VPC**. This is a foundational pattern for secure, production-ready AWS infrastructure.
