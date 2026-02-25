# 🌟 Task 14 - Provision IAM User with local-exec Provisioner Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is experimenting with **Terraform provisioners** to automate post-creation activities. The objective is to create an IAM user and use a `local-exec` provisioner to log a customized confirmation message to a local file on the runner machine immediately after the user is successfully provisioned.

**Requirements:**
- Create an **IAM user** named **`iamuser_siva`**
- Implement a **`local-exec` provisioner** within the IAM user resource to:
  - Log the message: `KKE iamuser_siva has been created successfully!`
  - Save the log to: `/home/bob/terraform/KKE_user_created.log`
- Define the user resource in a single **`main.tf`** file
- Use **`variables.tf`** to declare:
  - `KKE_USER_NAME` — name of the IAM user
- Use **`terraform.tfvars`** to provide the actual user name
- Use **`outputs.tf`** to export:
  - `kke_iam_user_name` — name of the created IAM user
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Provision an IAM user while demonstrating the use of Terraform's `local-exec` provisioner to execute local shell commands as part of the resource lifecycle.

💡 **Note:** Provisioners should generally be used as a **last resort**. While powerful for quick logging or triggering local scripts, they break Terraform's declarative model and can make the execution harder to track or roll back.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (IAM is global)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- IAM User (`iamuser_siva`)
- Local Log File (`KKE_user_created.log`) — created via shell command

**Working Directory:** `/home/bob/terraform`

**Provisioner Flow:**
```
Terraform Apply
        │
        ▼
Create aws_iam_user (iamuser_siva)
        │
        ▼ (Creation success)
Execute local-exec provisioner
        │
        ▼ (Shell command)
echo '...' > KKE_user_created.log
```

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_iam_user`:** The primary AWS resource being managed.
- **`provisioner "local-exec"`:** A block inside the resource that executes on the machine running Terraform.
- **Variable Interpolation:** The provisioner command uses `${var.KKE_USER_NAME}` to ensure the log message matches the actual resource name.
- **`variables.tf` & `terraform.tfvars`:** standard practices for keeping configuration clean and reusable.

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # IAM user resource + local-exec provisioner
├── variables.tf       # Variable declaration (KKE_USER_NAME)
├── terraform.tfvars   # User name assignment ("iamuser_siva")
└── outputs.tf         # Output for verification
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
variable "KKE_USER_NAME" {
  type = string
}
EOF
```

---

### Step 3: Create Terraform Variables File

Create `terraform.tfvars`:

```bash
cat > terraform.tfvars << 'EOF'
KKE_USER_NAME = "iamuser_siva"
EOF
```

---

### Step 4: Create Main Configuration File

Create `main.tf` including the `local-exec` provisioner:

```bash
cat > main.tf << 'EOF'
resource "aws_iam_user" "kke_user" {
  name = var.KKE_USER_NAME

  # Execute local command after resource creation
  provisioner "local-exec" {
    command = "echo 'KKE ${var.KKE_USER_NAME} has been created successfully!' > /home/bob/terraform/KKE_user_created.log"
  }
}
EOF
```

**Configuration Breakdown:**
- `command`: The specific shell command to run. It uses redirection (`>`) to write into the specified absolute path.
- **Lifecycle:** By default, provisioners are "creation-time" provisioners. They only run when the resource is created, not on subsequent updates.

---

### Step 5: Create Outputs Configuration File

Create `outputs.tf`:

```bash
cat > outputs.tf << 'EOF'
output "kke_iam_user_name" {
  value = aws_iam_user.kke_user.name
}
EOF
```

---

### Step 6: Initialize, Validate, and Apply

```bash
# Initialize
terraform init

# Validate
terraform validate

# Format
terraform fmt

# Apply
terraform apply -auto-approve
```

**Expected Provisioner Output:**
```
aws_iam_user.kke_user: Creating...
aws_iam_user.kke_user: Provisioning with 'local-exec'...
aws_iam_user.kke_user (local-exec): Executing: ["/bin/sh" "-c" "echo 'KKE iamuser_siva has been created successfully!' > /home/bob/terraform/KKE_user_created.log"]
aws_iam_user.kke_user: Creation complete after 1s [id=iamuser_siva]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

---

### Step 7: Verify Log File Creation

Check the content of the log file:

```bash
cat /home/bob/terraform/KKE_user_created.log
```

**Expected Output:**
```
KKE iamuser_siva has been created successfully!
```

---

### Step 8: Final State Verification (Critical)

```bash
terraform plan
```

**Expected Output:**
```
No changes. Your infrastructure matches the configuration.
```

✅ **Task complete.**

---

## 🔍 Code Analysis

### Creation-Time vs. Destroy-Time Provisioners

| Type | When it runs | Usage |
|------|--------------|-------|
| **Creation-Time** | When resource is created | Initial setup, registration, logging ✅ |
| **Destroy-Time** | Before resource is deleted | Deregistration, cleanup ❌ |

### Provisioner Failure Behavior
By default, if a provisioner fails (e.g., permission error writing to the log file), Terraform treats the entire resource creation as a **failure** and marks the resource as **tainted**. This means it will be destroyed and recreated on the next `apply`.

---

## 🎯 Task Completion Summary

### Resources Created
- **IAM User:** `iamuser_siva`
- **Local File:** `KKE_user_created.log`

### Task Completion Checklist
- [x] ✅ Provisioned IAM user `iamuser_siva`.
- [x] ✅ Successfully implemented `local-exec` provisioner.
- [x] ✅ Confirmation message logged to the correct path.
- [x] ✅ Variables and `tfvars` used correctly.
- [x] ✅ Output verified.
- [x] ✅ Final `terraform plan` returns **"No changes"**.
