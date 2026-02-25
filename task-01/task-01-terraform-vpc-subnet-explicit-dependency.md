# 🌟 Task 01 - Define Explicit Resource Dependencies Between AWS VPC and Subnet Using Terraform

## 📌 Task Description

To ensure **proper resource provisioning order** and maintain infrastructure reliability, the **DevOps team** wants to explicitly define the dependency between an AWS VPC (Virtual Private Cloud) and a Subnet. In complex infrastructure deployments, explicit dependencies help Terraform understand the correct creation and deletion order of resources, preventing race conditions and ensuring successful provisioning.

This task focuses on creating a VPC and Subnet with an explicit dependency relationship using Terraform's `depends_on` argument, implementing proper variable management, and providing outputs for infrastructure tracking.

**Requirements:**
- Create a **VPC** named **`datacenter-vpc`** with proper CIDR block configuration
- Create a **Subnet** named **`datacenter-subnet`** within the VPC
- Ensure the Subnet uses the **`depends_on`** argument to explicitly depend on the VPC resource
- Implement **variable-based configuration** for resource names
- Create **four separate Terraform files**: `main.tf`, `variables.tf`, `terraform.tfvars`, and `outputs.tf`
- Output the VPC and Subnet names for verification
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes"** after applying the configuration

👉 **Your task:** Create a VPC and Subnet with explicit dependency management using Terraform best practices, including proper variable management and output configuration.

💡 **Note:** Explicit dependencies using `depends_on` ensure predictable resource creation order, which is crucial for complex infrastructure where implicit dependencies may not capture all relationships.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)
**Provider:** AWS (Amazon Web Services)
**Resources:** 
- AWS VPC (Virtual Private Cloud) with /16 CIDR block
- AWS Subnet within VPC with /24 CIDR block and explicit dependency
- Variable-driven configuration for maintainability
- Output values for infrastructure verification

**Working Directory:** `/home/bob/terraform`

**Network Configuration:**
- **VPC CIDR Block:** `10.0.0.0/16` (65,536 IP addresses)
- **Subnet CIDR Block:** `10.0.1.0/24` (256 IP addresses)
- **Availability Zone:** `us-east-1a`

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **VPC Resource (`aws_vpc`):** Main virtual network environment with isolated IP space
- **Subnet Resource (`aws_subnet`):** Subdivision of VPC for resource placement
- **Explicit Dependency:** `depends_on` argument ensures proper provisioning order
- **Variable Management:** Centralized variable definitions and value assignments
- **Output Configuration:** Infrastructure state visibility and verification

### 🎯 Implementation Strategy
1. Define variables for VPC and Subnet names in `variables.tf`
2. Assign values to variables in `terraform.tfvars`
3. Create VPC resource in `main.tf` with variable-based naming
4. Create Subnet resource with explicit `depends_on` dependency
5. Configure outputs in `outputs.tf` for name verification
6. Initialize, apply, and verify the infrastructure

### 📁 File Structure
```
/home/bob/terraform/
├── main.tf           # Primary resource definitions
├── variables.tf      # Variable declarations
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

**Expected Output:**
```
/home/bob/terraform
```

**Verification:**
```bash
pwd
```

---

### Step 2: Create Variables Definition File

Create `variables.tf` to define input variables:

```bash
cat > variables.tf << 'EOF'
variable "KKE_VPC_NAME" {
  type        = string
  description = "Name of the VPC"
}

variable "KKE_SUBNET_NAME" {
  type        = string
  description = "Name of the Subnet"
}
EOF
```

**Purpose:** Define the input variables that will be used throughout the configuration.

**Configuration Breakdown:**
- `variable "KKE_VPC_NAME"` - Declares variable for VPC name
- `type = string` - Specifies the variable type as string
- `description` - Provides documentation for the variable purpose
- **Best Practice:** Variable names use uppercase with underscores for environment/configuration variables

**File Content (`variables.tf`):**
```hcl
variable "KKE_VPC_NAME" {
  type        = string
  description = "Name of the VPC"
}

variable "KKE_SUBNET_NAME" {
  type        = string
  description = "Name of the Subnet"
}
```

---

### Step 3: Create Variable Values File

Create `terraform.tfvars` to assign values to variables:

```bash
cat > terraform.tfvars << 'EOF'
KKE_VPC_NAME    = "datacenter-vpc"
KKE_SUBNET_NAME = "datacenter-subnet"
EOF
```

**Purpose:** Provide actual values for the defined variables, separating configuration from code.

**Configuration Breakdown:**
- `KKE_VPC_NAME = "datacenter-vpc"` - Assigns the VPC name as required
- `KKE_SUBNET_NAME = "datacenter-subnet"` - Assigns the Subnet name as required
- **Best Practice:** Using `.tfvars` files allows environment-specific configurations without modifying code

**File Content (`terraform.tfvars`):**
```hcl
KKE_VPC_NAME    = "datacenter-vpc"
KKE_SUBNET_NAME = "datacenter-subnet"
```

---

### Step 4: Create Main Configuration File

Create `main.tf` with VPC and Subnet resources:

```bash
cat > main.tf << 'EOF'
# Create VPC
resource "aws_vpc" "datacenter_vpc" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = var.KKE_VPC_NAME
  }
}

# Create Subnet with explicit depends_on
resource "aws_subnet" "datacenter_subnet" {
  vpc_id            = aws_vpc.datacenter_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  
  depends_on = [
    aws_vpc.datacenter_vpc
  ]
  
  tags = {
    Name = var.KKE_SUBNET_NAME
  }
}
EOF
```

**Purpose:** Define the primary AWS resources (VPC and Subnet) with explicit dependency relationship.

**Configuration Breakdown:**

**VPC Resource (`aws_vpc.datacenter_vpc`):**
- `cidr_block = "10.0.0.0/16"` - Defines the IP address range for the VPC (65,536 addresses)
- `tags.Name = var.KKE_VPC_NAME` - Uses variable for dynamic naming
- **Implicit Output:** Resource ID available as `aws_vpc.datacenter_vpc.id`

**Subnet Resource (`aws_subnet.datacenter_subnet`):**
- `vpc_id = aws_vpc.datacenter_vpc.id` - References the VPC (implicit dependency)
- `cidr_block = "10.0.1.0/24"` - Defines subnet IP range (256 addresses, must be within VPC CIDR)
- `availability_zone = "us-east-1a"` - Specifies physical location for the subnet
- `depends_on = [aws_vpc.datacenter_vpc]` - **Explicit dependency** ensures VPC is created first
- `tags.Name = var.KKE_SUBNET_NAME` - Uses variable for dynamic naming

**Dependency Types:**
| Type | Example | Purpose |
|------|---------|---------|
| **Implicit** | `vpc_id = aws_vpc.datacenter_vpc.id` | Terraform automatically detects from resource references |
| **Explicit** | `depends_on = [aws_vpc.datacenter_vpc]` | Manually specified for additional control |

**File Content (`main.tf`):**
```hcl
# Create VPC
resource "aws_vpc" "datacenter_vpc" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = var.KKE_VPC_NAME
  }
}

# Create Subnet with explicit depends_on
resource "aws_subnet" "datacenter_subnet" {
  vpc_id            = aws_vpc.datacenter_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  
  depends_on = [
    aws_vpc.datacenter_vpc
  ]
  
  tags = {
    Name = var.KKE_SUBNET_NAME
  }
}
```

---

### Step 5: Create Outputs Configuration File

Create `outputs.tf` to export resource information:

```bash
cat > outputs.tf << 'EOF'
output "kke_vpc_name" {
  value = aws_vpc.datacenter_vpc.tags["Name"]
}

output "kke_subnet_name" {
  value = aws_subnet.datacenter_subnet.tags["Name"]
}
EOF
```

**Purpose:** Define output values for verification and reference by other systems or modules.

**Configuration Breakdown:**
- `output "kke_vpc_name"` - Declares an output named `kke_vpc_name`
- `value = aws_vpc.datacenter_vpc.tags["Name"]` - Extracts the VPC name from tags
- **Output Access:** Values available after `terraform apply` via `terraform output`

**File Content (`outputs.tf`):**
```hcl
output "kke_vpc_name" {
  value = aws_vpc.datacenter_vpc.tags["Name"]
}

output "kke_subnet_name" {
  value = aws_subnet.datacenter_subnet.tags["Name"]
}
```

---

### Step 6: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Purpose:** Confirm all required files are created correctly.

**Expected Output:**
```
total 16
drwxr-xr-x 2 bob bob 4096 Oct 25 16:00 .
drwxr-xr-x 5 bob bob 4096 Oct 25 15:55 ..
-rw-r--r-- 1 bob bob  456 Oct 25 16:00 main.tf
-rw-r--r-- 1 bob bob  234 Oct 25 16:00 outputs.tf
-rw-r--r-- 1 bob bob   89 Oct 25 16:00 terraform.tfvars
-rw-r--r-- 1 bob bob  178 Oct 25 16:00 variables.tf
```

**Success Indicators:**
- ✅ All four files present: `main.tf`, `variables.tf`, `terraform.tfvars`, `outputs.tf`
- ✅ Files have appropriate permissions (readable)
- ✅ Files created in correct directory

---

### Step 7: Initialize Terraform

```bash
terraform init
```

**Purpose:** Initialize the Terraform working directory, download AWS provider, and prepare backend.

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
- ✅ Backend initialization successful
- ✅ AWS provider downloaded and installed
- ✅ `.terraform` directory created
- ✅ `.terraform.lock.hcl` file generated (locks provider versions)

**Created Files/Directories:**
```
/home/bob/terraform/
├── .terraform/          # Provider plugins directory
├── .terraform.lock.hcl  # Provider version lock file
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

**Purpose:** Verify the syntax and internal consistency of Terraform configuration files.

**Expected Output:**
```
Success! The configuration is valid.
```

**What's Validated:**
- ✅ HCL syntax correctness
- ✅ Variable references are valid
- ✅ Resource attribute names are correct
- ✅ Resource dependencies are properly defined

---

### Step 9: Format Configuration Files

```bash
terraform fmt
```

**Purpose:** Automatically format Terraform configuration files to canonical style.

**Expected Output:**
```
main.tf
variables.tf
terraform.tfvars
outputs.tf
```

**Formatting Applied:**
- ✅ Consistent indentation (2 spaces)
- ✅ Proper alignment of equals signs
- ✅ Consistent blank line usage
- ✅ Standardized code style

---

### Step 10: Review Execution Plan

```bash
terraform plan
```

**Purpose:** Preview the changes Terraform will make to infrastructure before applying.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_subnet.datacenter_subnet will be created
  + resource "aws_subnet" "datacenter_subnet" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.1.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Name" = "datacenter-subnet"
        }
      + tags_all                                       = {
          + "Name" = "datacenter-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # aws_vpc.datacenter_vpc will be created
  + resource "aws_vpc" "datacenter_vpc" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_dns_hostnames                 = (known after apply)
      + enable_dns_support                   = true
      + enable_network_address_usage_metrics = (known after apply)
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Name" = "datacenter-vpc"
        }
      + tags_all                             = {
          + "Name" = "datacenter-vpc"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + kke_subnet_name = "datacenter-subnet"
  + kke_vpc_name    = "datacenter-vpc"

───────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
```

**Plan Summary:**
- **Resources to Create:** 2 (VPC + Subnet)
- **Resources to Change:** 0
- **Resources to Destroy:** 0
- **Outputs to Create:** 2 (VPC name + Subnet name)

**Success Indicators:**
- ✅ Plan shows 2 resources to be created
- ✅ VPC will be created with correct CIDR block and name
- ✅ Subnet will be created with explicit dependency
- ✅ Output values configured correctly

---

### Step 11: Apply Configuration

```bash
terraform apply -auto-approve
```

**Purpose:** Execute the planned changes to create the VPC and Subnet infrastructure.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_subnet.datacenter_subnet will be created
  + resource "aws_subnet" "datacenter_subnet" {
      + arn                                            = (known after apply)
      + availability_zone                              = "us-east-1a"
      + cidr_block                                     = "10.0.1.0/24"
      + id                                             = (known after apply)
      + tags                                           = {
          + "Name" = "datacenter-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # aws_vpc.datacenter_vpc will be created
  + resource "aws_vpc" "datacenter_vpc" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + id                                   = (known after apply)
      + tags                                 = {
          + "Name" = "datacenter-vpc"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + kke_subnet_name = "datacenter-subnet"
  + kke_vpc_name    = "datacenter-vpc"

aws_vpc.datacenter_vpc: Creating...
aws_vpc.datacenter_vpc: Creation complete after 2s [id=vpc-0a1b2c3d4e5f6g7h8]
aws_subnet.datacenter_subnet: Creating...
aws_subnet.datacenter_subnet: Creation complete after 1s [id=subnet-1a2b3c4d5e6f7g8h9]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

kke_subnet_name = "datacenter-subnet"
kke_vpc_name = "datacenter-vpc"
```

**Success Indicators:**
- ✅ VPC created successfully (note: VPC created first)
- ✅ Subnet created after VPC (explicit dependency honored)
- ✅ Both resources have unique IDs
- ✅ Output values displayed correctly
- ✅ No errors during creation

**Creation Order Verification:**
1. **First:** `aws_vpc.datacenter_vpc` - Created due to explicit dependency
2. **Second:** `aws_subnet.datacenter_subnet` - Waited for VPC completion

---

### Step 12: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Purpose:** Confirm that infrastructure matches configuration with no pending changes.

**Expected Output:**
```
aws_vpc.datacenter_vpc: Refreshing state... [id=vpc-0a1b2c3d4e5f6g7h8]
aws_subnet.datacenter_subnet: Refreshing state... [id=subnet-1a2b3c4d5e6f7g8h9]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ Both resources refreshed successfully
- ✅ No drift detected between state and actual infrastructure
- ✅ Configuration is complete and accurate

**⚠️ Important:** This verification step is **required** before task submission. The "No changes" message confirms successful completion.

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Purpose:** List all resources managed by Terraform in the current state.

**Expected Output:**
```
aws_subnet.datacenter_subnet
aws_vpc.datacenter_vpc
```

**Success Indicators:**
- ✅ Both resources present in state
- ✅ Resource names match configuration

---

### Step 2: Check VPC Details

```bash
terraform state show aws_vpc.datacenter_vpc
```

**Purpose:** Display detailed information about the VPC resource.

**Expected Output:**
```
# aws_vpc.datacenter_vpc:
resource "aws_vpc" "datacenter_vpc" {
    arn                              = "arn:aws:ec2:us-east-1:123456789012:vpc/vpc-0a1b2c3d4e5f6g7h8"
    cidr_block                       = "10.0.0.0/16"
    default_network_acl_id           = "acl-0a1b2c3d4e5f6g7h8"
    default_route_table_id           = "rtb-0a1b2c3d4e5f6g7h8"
    default_security_group_id        = "sg-0a1b2c3d4e5f6g7h8"
    dhcp_options_id                  = "dopt-0a1b2c3d4e5f6g7h8"
    enable_dns_hostnames             = false
    enable_dns_support               = true
    enable_network_address_usage_metrics = false
    id                               = "vpc-0a1b2c3d4e5f6g7h8"
    instance_tenancy                 = "default"
    main_route_table_id              = "rtb-0a1b2c3d4e5f6g7h8"
    owner_id                         = "123456789012"
    tags                             = {
        "Name" = "datacenter-vpc"
    }
    tags_all                         = {
        "Name" = "datacenter-vpc"
    }
}
```

**Verification Points:**
- ✅ CIDR block is `10.0.0.0/16`
- ✅ Name tag is `datacenter-vpc`
- ✅ Resource has valid ID and ARN

---

### Step 3: Check Subnet Details

```bash
terraform state show aws_subnet.datacenter_subnet
```

**Purpose:** Display detailed information about the Subnet resource.

**Expected Output:**
```
# aws_subnet.datacenter_subnet:
resource "aws_subnet" "datacenter_subnet" {
    arn                                            = "arn:aws:ec2:us-east-1:123456789012:subnet/subnet-1a2b3c4d5e6f7g8h9"
    availability_zone                              = "us-east-1a"
    availability_zone_id                           = "use1-az1"
    cidr_block                                     = "10.0.1.0/24"
    enable_dns64                                   = false
    enable_resource_name_dns_a_record_on_launch    = false
    enable_resource_name_dns_aaaa_record_on_launch = false
    id                                             = "subnet-1a2b3c4d5e6f7g8h9"
    ipv6_native                                    = false
    map_public_ip_on_launch                        = false
    owner_id                                       = "123456789012"
    private_dns_hostname_type_on_launch            = "ip-name"
    tags                                           = {
        "Name" = "datacenter-subnet"
    }
    tags_all                                       = {
        "Name" = "datacenter-subnet"
    }
    vpc_id                                         = "vpc-0a1b2c3d4e5f6g7h8"
}
```

**Verification Points:**
- ✅ CIDR block is `10.0.1.0/24` (within VPC range)
- ✅ VPC ID matches the created VPC
- ✅ Availability zone is `us-east-1a`
- ✅ Name tag is `datacenter-subnet`

---

### Step 4: View Output Values

```bash
terraform output
```

**Purpose:** Display all configured output values.

**Expected Output:**
```
kke_subnet_name = "datacenter-subnet"
kke_vpc_name = "datacenter-vpc"
```

**Individual Output Access:**
```bash
# Get specific output value
terraform output kke_vpc_name
terraform output kke_subnet_name
```

**Expected Output:**
```
"datacenter-vpc"
"datacenter-subnet"
```

---

### Step 5: AWS CLI Verification (Optional)

```bash
# Verify VPC exists in AWS
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=datacenter-vpc"

# Verify Subnet exists in AWS
aws ec2 describe-subnets --filters "Name=tag:Name,Values=datacenter-subnet"
```

**Purpose:** Confirm resources exist in AWS and match Terraform state.

---

## 🔍 Code Analysis

### Complete Solution Structure

**File Organization:**
```
/home/bob/terraform/
├── main.tf              # Primary resource definitions (VPC + Subnet)
├── variables.tf         # Variable declarations
├── terraform.tfvars     # Variable value assignments
└── outputs.tf           # Output value definitions
```

### Dependency Analysis

**Implicit vs. Explicit Dependencies:**

```hcl
# Implicit Dependency (automatically detected)
resource "aws_subnet" "datacenter_subnet" {
  vpc_id = aws_vpc.datacenter_vpc.id  # Reference creates implicit dependency
  # ...
}

# Explicit Dependency (manually specified)
resource "aws_subnet" "datacenter_subnet" {
  depends_on = [
    aws_vpc.datacenter_vpc  # Forces creation order
  ]
  # ...
}
```

**Dependency Comparison:**

| Aspect | Implicit Dependency | Explicit Dependency |
|--------|-------------------|-------------------|
| **Detection** | Automatic (via resource references) | Manual (via `depends_on`) |
| **Example** | `vpc_id = aws_vpc.datacenter_vpc.id` | `depends_on = [aws_vpc.datacenter_vpc]` |
| **Use Case** | Most resource relationships | Hidden dependencies, API ordering |
| **Best Practice** | Prefer implicit when possible | Use when implicit isn't sufficient |

**Why Both in This Task:**
- **Implicit:** `vpc_id` reference ensures Terraform knows Subnet needs VPC ID
- **Explicit:** `depends_on` demonstrates explicit dependency management for the task requirement

### Variable Flow Diagram

```
variables.tf (Declaration)
    ↓
terraform.tfvars (Value Assignment)
    ↓
main.tf (Variable Usage: var.KKE_VPC_NAME)
    ↓
AWS Resource (Applied as Tag)
    ↓
outputs.tf (Value Extraction)
    ↓
Terminal Output (Display)
```

### CIDR Block Relationship

```
VPC:    10.0.0.0/16    (10.0.0.0 - 10.0.255.255)    65,536 IPs
         ↓
Subnet: 10.0.1.0/24    (10.0.1.0 - 10.0.1.255)      256 IPs
```

**CIDR Rules:**
- ✅ Subnet CIDR must be within VPC CIDR range
- ✅ Subnet CIDR must be more specific (larger prefix: /24 > /16)
- ✅ Subnets cannot overlap within the same VPC

---

## 🧪 Testing and Validation

### Test 1: Configuration Syntax Validation

```bash
terraform validate
```

**Expected:** `Success! The configuration is valid.`

### Test 2: Plan Consistency Check

```bash
terraform plan
```

**Expected:** `No changes. Your infrastructure matches the configuration.`

### Test 3: State Consistency Verification

```bash
# Check state file integrity
terraform show

# Verify resource count
terraform state list | wc -l
```

**Expected:** 2 resources listed

### Test 4: Dependency Order Verification

```bash
# View resource creation order from apply output
terraform apply -auto-approve 2>&1 | grep "Creating\|Creation complete"
```

**Expected Order:**
1. VPC creates first
2. Subnet creates second (after VPC)

### Test 5: Output Value Verification

```bash
# Verify outputs match variable values
terraform output -json | jq -r '.kke_vpc_name.value'
terraform output -json | jq -r '.kke_subnet_name.value'
```

**Expected:**
```
datacenter-vpc
datacenter-subnet
```

### Test 6: Resource Tag Verification

```bash
# Verify VPC tags
aws ec2 describe-vpcs --vpc-ids $(terraform output -raw vpc_id 2>/dev/null || terraform state show aws_vpc.datacenter_vpc | grep "id " | awk '{print $3}' | tr -d '"') --query 'Vpcs[0].Tags'

# Verify Subnet tags
aws ec2 describe-subnets --subnet-ids $(terraform state show aws_subnet.datacenter_subnet | grep "id " | awk '{print $3}' | tr -d '"') --query 'Subnets[0].Tags'
```

---

## 📚 Quick Reference

### Complete Solution Files

**`main.tf`:**
```hcl
# Create VPC
resource "aws_vpc" "datacenter_vpc" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = var.KKE_VPC_NAME
  }
}

# Create Subnet with explicit depends_on
resource "aws_subnet" "datacenter_subnet" {
  vpc_id            = aws_vpc.datacenter_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  
  depends_on = [
    aws_vpc.datacenter_vpc
  ]
  
  tags = {
    Name = var.KKE_SUBNET_NAME
  }
}
```

**`variables.tf`:**
```hcl
variable "KKE_VPC_NAME" {
  type        = string
  description = "Name of the VPC"
}

variable "KKE_SUBNET_NAME" {
  type        = string
  description = "Name of the Subnet"
}
```

**`terraform.tfvars`:**
```hcl
KKE_VPC_NAME    = "datacenter-vpc"
KKE_SUBNET_NAME = "datacenter-subnet"
```

**`outputs.tf`:**
```hcl
output "kke_vpc_name" {
  value = aws_vpc.datacenter_vpc.tags["Name"]
}

output "kke_subnet_name" {
  value = aws_subnet.datacenter_subnet.tags["Name"]
}
```

### Essential Commands

```bash
# Navigate to working directory
cd /home/bob/terraform

# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt

# Review plan
terraform plan

# Apply configuration
terraform apply -auto-approve

# Verify no changes (REQUIRED)
terraform plan

# View outputs
terraform output

# List resources
terraform state list

# View resource details
terraform state show aws_vpc.datacenter_vpc
terraform state show aws_subnet.datacenter_subnet
```

### One-Line Complete Setup

```bash
cd /home/bob/terraform && terraform init && terraform apply -auto-approve && terraform plan
```

---

## 🛠️ Troubleshooting

### Common Issues

**Issue 1: Variable Not Found**
- **Symptoms:** Error: `Reference to undeclared input variable`
- **Solution:** Ensure variable is declared in `variables.tf`
```bash
# Check if variable exists
grep "KKE_VPC_NAME" variables.tf
```

**Issue 2: CIDR Block Overlap**
- **Symptoms:** Error: `InvalidSubnet.Conflict` or `CIDR block overlap`
- **Solution:** Ensure subnet CIDR is within VPC CIDR range
```hcl
# Correct configuration
VPC:    cidr_block = "10.0.0.0/16"
Subnet: cidr_block = "10.0.1.0/24"  # Must be within 10.0.0.0/16
```

**Issue 3: Availability Zone Not Available**
- **Symptoms:** Error: `InvalidInput` or AZ not available
- **Solution:** Use a valid AZ for your region
```bash
# List available AZs
aws ec2 describe-availability-zones --region us-east-1
```

**Issue 4: Terraform State Lock**
- **Symptoms:** Error: `Error acquiring the state lock`
- **Solution:** Wait for lock to release or force unlock if stuck
```bash
# Force unlock (use carefully)
terraform force-unlock <lock-id>
```

**Issue 5: Provider Authentication**
- **Symptoms:** Error: `No valid credential sources found`
- **Solution:** Configure AWS credentials
```bash
# Set AWS credentials
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"

# Or configure via AWS CLI
aws configure
```

**Issue 6: "No changes" Check Fails**
- **Symptoms:** `terraform plan` shows changes after apply
- **Solution:** Verify no manual changes made to AWS resources
```bash
# Refresh state and check again
terraform refresh
terraform plan
```

**Issue 7: Output Not Displaying**
- **Symptoms:** Output values not shown after apply
- **Solution:** Explicitly request outputs
```bash
# Display outputs
terraform output
terraform output kke_vpc_name
```

**Issue 8: Dependency Not Honored**
- **Symptoms:** Resources created in wrong order
- **Solution:** Verify `depends_on` syntax
```hcl
# Correct syntax
depends_on = [
  aws_vpc.datacenter_vpc
]

# Incorrect syntax (missing brackets)
depends_on = aws_vpc.datacenter_vpc  # WRONG
```

---

## 💡 Best Practices Applied

### Configuration Management
- **✅ Separation of Concerns:** Variables, resources, and outputs in separate files
- **✅ Variable-Driven:** Dynamic naming using variables
- **✅ DRY Principle:** Values defined once, referenced multiple times
- **✅ Documentation:** Comments explain resource purposes

### Dependency Management
- **✅ Explicit Dependencies:** `depends_on` for clear ordering
- **✅ Implicit Dependencies:** Resource references for natural relationships
- **✅ Predictable Provisioning:** Guaranteed resource creation order

### Code Quality
- **✅ Formatting:** Consistent style using `terraform fmt`
- **✅ Validation:** Syntax checking with `terraform validate`
- **✅ Verification:** State comparison with `terraform plan`

### Output Management
- **✅ Meaningful Names:** Clear, descriptive output names
- **✅ Value Extraction:** Accessing nested attributes correctly
- **✅ Documentation:** Output values for infrastructure tracking

---

## 🚀 Production Considerations

### Enhanced VPC Configuration

```hcl
# Production-grade VPC with additional features
resource "aws_vpc" "datacenter_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true   # Enable DNS hostnames
  enable_dns_support   = true   # Enable DNS resolution
  
  tags = {
    Name        = var.KKE_VPC_NAME
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# Production-grade Subnet with additional features
resource "aws_subnet" "datacenter_subnet" {
  vpc_id                  = aws_vpc.datacenter_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = false  # Private subnet
  
  depends_on = [
    aws_vpc.datacenter_vpc
  ]
  
  tags = {
    Name        = var.KKE_SUBNET_NAME
    Environment = var.environment
    Tier        = "Private"
    ManagedBy   = "Terraform"
  }
}
```

### Multi-AZ Subnet Configuration

```hcl
# Multiple subnets for high availability
resource "aws_subnet" "datacenter_subnets" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.datacenter_vpc.id
  cidr_block        = cidrsubnet(aws_vpc.datacenter_vpc.cidr_block, 8, count.index)
  availability_zone = var.availability_zones[count.index]
  
  depends_on = [
    aws_vpc.datacenter_vpc
  ]
  
  tags = {
    Name = "${var.KKE_SUBNET_NAME}-${count.index + 1}"
  }
}
```

### Enhanced Outputs

```hcl
# Additional outputs for production use
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.datacenter_vpc.id
}

output "vpc_arn" {
  description = "ARN of the VPC"
  value       = aws_vpc.datacenter_vpc.arn
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.datacenter_vpc.cidr_block
}

output "subnet_id" {
  description = "ID of the Subnet"
  value       = aws_subnet.datacenter_subnet.id
}

output "subnet_arn" {
  description = "ARN of the Subnet"
  value       = aws_subnet.datacenter_subnet.arn
}

output "subnet_cidr_block" {
  description = "CIDR block of the Subnet"
  value       = aws_subnet.datacenter_subnet.cidr_block
}

output "availability_zone" {
  description = "Availability zone of the Subnet"
  value       = aws_subnet.datacenter_subnet.availability_zone
}
```

### Variable Validation

```hcl
# Add validation to variables
variable "KKE_VPC_NAME" {
  type        = string
  description = "Name of the VPC"
  
  validation {
    condition     = length(var.KKE_VPC_NAME) > 0 && length(var.KKE_VPC_NAME) <= 255
    error_message = "VPC name must be between 1 and 255 characters."
  }
}

variable "KKE_SUBNET_NAME" {
  type        = string
  description = "Name of the Subnet"
  
  validation {
    condition     = length(var.KKE_SUBNET_NAME) > 0 && length(var.KKE_SUBNET_NAME) <= 255
    error_message = "Subnet name must be between 1 and 255 characters."
  }
}
```

### Remote State Backend

```hcl
# Configure remote state storage (add to main.tf or create backend.tf)
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "vpc-subnet/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

### Provider Version Constraints

```hcl
# Specify provider version (add to main.tf or create versions.tf)
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      ManagedBy = "Terraform"
      Project   = "Datacenter-Infrastructure"
    }
  }
}
```

---

## 📖 Learning Outcomes

**Key Concepts Mastered:**
- **✅ Explicit Dependencies:** Using `depends_on` to control resource creation order
- **✅ Variable Management:** Separating declarations, values, and usage
- **✅ Output Configuration:** Extracting and displaying resource attributes
- **✅ File Organization:** Structuring Terraform projects for maintainability
- **✅ VPC Networking:** Understanding AWS VPC and Subnet relationships

**Terraform Features Used:**
- `resource` - Resource block definition
- `variable` - Input variable declaration
- `output` - Output value definition
- `depends_on` - Explicit dependency specification
- `var.*` - Variable reference syntax
- `tags` - Resource tagging for identification

**AWS Concepts Applied:**
- **VPC (Virtual Private Cloud):** Isolated network environment
- **CIDR Blocks:** IP address range specification
- **Subnets:** Network subdivisions within VPC
- **Availability Zones:** Physical datacenter locations
- **Resource Tags:** Metadata for resource organization

---

## 🎯 Task Completion Summary

✅ **Successfully Completed:**
- **VPC Creation** - Created `datacenter-vpc` with 10.0.0.0/16 CIDR block
- **Subnet Creation** - Created `datacenter-subnet` with 10.0.1.0/24 CIDR block
- **Explicit Dependency** - Implemented `depends_on` argument correctly
- **Variable Management** - Defined variables in `variables.tf`, assigned values in `terraform.tfvars`
- **Output Configuration** - Created outputs in `outputs.tf` for VPC and Subnet names
- **File Structure** - Used four separate files as required (`main.tf`, `variables.tf`, `terraform.tfvars`, `outputs.tf`)
- **Verification** - Confirmed `terraform plan` returns "No changes" message

**Final Status:** 🎉 **Task completed successfully with all requirements met**

### 📊 Resource Summary

| Resource | Name | CIDR Block | Availability Zone | Status |
|----------|------|------------|------------------|---------|
| **VPC** | datacenter-vpc | 10.0.0.0/16 | us-east-1 (region) | ✅ Created |
| **Subnet** | datacenter-subnet | 10.0.1.0/24 | us-east-1a | ✅ Created |

### 🔗 Dependency Chain

```
aws_vpc.datacenter_vpc (Created First)
         ↓
         │ (depends_on)
         ↓
aws_subnet.datacenter_subnet (Created Second)
```

### 📋 Verification Checklist

- [✅] All required files created (`main.tf`, `variables.tf`, `terraform.tfvars`, `outputs.tf`)
- [✅] VPC created with correct name and CIDR block
- [✅] Subnet created with correct name and CIDR block
- [✅] Explicit dependency (`depends_on`) implemented
- [✅] Variables properly declared and assigned
- [✅] Outputs correctly configured
- [✅] `terraform init` successful
- [✅] `terraform validate` passed
- [✅] `terraform plan` showed 2 resources to create
- [✅] `terraform apply` completed successfully
- [✅] **Final `terraform plan` returns "No changes"** ✨

**Infrastructure Foundation:** The VPC and Subnet infrastructure is now ready for additional resource deployment, with proper dependency management ensuring reliable provisioning order.

### 🔮 Infrastructure Ready For:
- **EC2 Instances:** Launch compute resources in the subnet
- **RDS Databases:** Deploy databases within the VPC
- **Load Balancers:** Create application load balancers
- **Security Groups:** Define network security rules
- **Route Tables:** Configure network routing
- **Internet Gateway:** Enable internet connectivity (if needed)
- **NAT Gateway:** Provide outbound internet for private resources

---

