# 🌟 Task 18 - Provision EC2 Instance and CloudWatch Alarm for CPU Monitoring Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is optimizing application performance monitoring by implementing automated alerts for resource utilization. The goal is to provision an EC2 instance and configure a **CloudWatch alarm** that monitors its CPU utilization. If the instance experiences high load (exceeding 90% for a sustained 5-minute period), the alarm will automatically trigger a notification to a pre-existing SNS topic, alerting the team to potential performance issues.

**Requirements:**
- Launch an **EC2 Instance** named **`datacenter-ec2`**:
  - AMI: **`ami-0c02fb55956c7d316`** (Ubuntu)
  - Instance Type: `t2.micro` (or appropriate)
- Create a **CloudWatch Alarm** named **`datacenter-alarm`**:
  - **Metric:** CPU Utilization
  - **Namespace:** `AWS/EC2`
  - **Statistic:** `Average`
  - **Threshold:** `>= 90%`
  - **Period:** 300 seconds (5 minutes)
  - **Evaluation Periods:** 1
  - **Comparison Operator:** `GreaterThanOrEqualToThreshold`
  - **Alarm Action:** Send notification to a specific SNS topic.
- Use an **SNS topic** named **`datacenter-sns-topic`** for notifications.
- All resources (EC2, SNS topic managed resource, CloudWatch alarm) must be in a single **`main.tf`** file.
- Use **`outputs.tf`** to export:
  - `KKE_instance_name`: The name tag of the EC2 instance.
  - `KKE_alarm_name`: The name of the CloudWatch alarm.
- The Terraform working directory is **`/home/bob/terraform`**.
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Build an automated monitoring and alerting stack using Terraform — ensuring that critical performance metrics for your compute resources are actively tracked and reported.

💡 **Note:** To trigger the alarm for a 5-minute period, the `period` must be set to `300` seconds and `evaluation_periods` to `1`. This ensures the alarm checks the average CPU over that specific window before changing state.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- EC2 Instance (`datacenter-ec2`) — Ubuntu-based compute
- SNS Topic (`datacenter-sns-topic`) — Notification channel
- CloudWatch Metric Alarm (`datacenter-alarm`) — CPU monitor

**Working Directory:** `/home/bob/terraform`

**Monitoring & Alerting Flow:**
```
[EC2 Instance]
      │
      │ (Metrics sent to CloudWatch every 5 min)
      ▼
[CloudWatch Metric Alarm]
      │
      │ Check: CPU > 90%?
      ▼ (YES)
[SNS Topic: datacenter-sns-topic]
      │
      ▼
[Subscribers (Email/SMS/PagerDuty)]
```

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_instance`:** Provisions the Ubuntu instance with the specific AMI and name tag.
- **`aws_sns_topic`:** Manages/References the notification topic.
- **`aws_cloudwatch_metric_alarm`:** The core monitoring logic. It uses the `InstanceId` dimension to target the metrics specifically from the `datacenter_ec2` resource.
- **Standard Layout:** Logical separation into `main.tf` and `outputs.tf`.

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # EC2 instance, SNS topic, and Alarm configuration
└── outputs.tf         # Outputs for instance and alarm names
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

---

### Step 2: Create Main Configuration File

Create `main.tf` including the EC2 instance, SNS topic reference, and CloudWatch alarm:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# SNS Topic (Managed Resource)
# ---------------------------
resource "aws_sns_topic" "sns_topic" {
  name = "datacenter-sns-topic"
}

# ---------------------------
# Launch EC2 Instance
# ---------------------------
resource "aws_instance" "datacenter_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"

  tags = {
    Name = "datacenter-ec2"
  }
}

# ---------------------------
# CloudWatch Alarm
# ---------------------------
resource "aws_cloudwatch_metric_alarm" "datacenter_alarm" {
  alarm_name          = "datacenter-alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300
  statistic           = "Average"
  threshold           = 90

  dimensions = {
    InstanceId = aws_instance.datacenter_ec2.id
  }

  alarm_actions = [aws_sns_topic.sns_topic.arn]
}
EOF
```

**Configuration Breakdown:**
- **Period (300):** Defines the granularity of the metric check (5 minutes).
- **Evaluation Periods (1):** The number of consecutive periods the threshold must be breached to trigger the alarm.
- **Dimensions:** Filters the `CPUUtilization` metric by the specific `InstanceId` of our new EC2 instance.
- **Alarm Actions:** Points to the SNS Topic ARN to trigger a notification when the state moves to `ALARM`.

---

### Step 3: Create Outputs Configuration File

Create `outputs.tf`:

```bash
cat > outputs.tf << 'EOF'
output "KKE_instance_name" {
  value = aws_instance.datacenter_ec2.tags["Name"]
}

output "KKE_alarm_name" {
  value = aws_cloudwatch_metric_alarm.datacenter_alarm.alarm_name
}
EOF
```

---

### Step 4: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

**Expected Output:**
```
total 12
drwxr-xr-x 2 bob bob 4096 Feb 26 20:30 .
drwxr-xr-x 5 bob bob 4096 Feb 26 20:25 ..
-rw-r--r-- 1 bob bob  845 Feb 26 20:30 main.tf
-rw-r--r-- 1 bob bob  225 Feb 26 20:30 outputs.tf
```

---

### Step 5: Initialize, Validate, and Apply

```bash
# Initialize Terraform
terraform init

# Validate configuration syntax
terraform validate

# Format the code
terraform fmt

# Apply the execution plan
terraform apply -auto-approve
```

**Expected Success Message:**
```
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

KKE_alarm_name = "datacenter-alarm"
KKE_instance_name = "datacenter-ec2"
```

---

### Step 6: Final Verification (Critical)

```bash
terraform plan
```

**Expected Output:**
```
No changes. Your infrastructure matches the configuration.
```

✅ **Task complete.**

---

## ✅ Verification Steps

### Step 1: Verify State List

```bash
terraform state list
```

**Expected Output:**
```
aws_cloudwatch_metric_alarm.datacenter_alarm
aws_instance.datacenter_ec2
aws_sns_topic.sns_topic
```

---

### Step 2: Inspect CloudWatch Alarm in Console/State

```bash
terraform state show aws_cloudwatch_metric_alarm.datacenter_alarm
```

**Verify:** Check `threshold = 90`, `period = 300`, and `comparison_operator`.

---

### Step 3: Verify SNS Topic Link

```bash
# Check that the alarm action points to the correct topic
aws cloudwatch describe-alarms --alarm-names datacenter-alarm --query 'MetricAlarms[0].AlarmActions'
```

---

## 🔍 Code Analysis

### Dynamic Resource Association
The CloudWatch alarm uses `aws_instance.datacenter_ec2.id`. This creates an **implicit dependency**. Terraform will:
1. Create the SNS topic.
2. Launch the EC2 instance to obtain its `InstanceId`.
3. Finally, create the CloudWatch Alarm with that ID.

### Metric Granularity
Standard monitoring for EC2 is free and reports metrics every 5 minutes. Setting the `period` to 300 matches this free-tier granularity. Setting it lower (e.g., 60 seconds) would require enabling **Detailed Monitoring** on the EC2 instance, which incurs additional costs.

---

## 🎯 Task Completion Summary

### Resources Created

| Resource | Value |
|----------|-------|
| EC2 Instance | `datacenter-ec2` |
| CloudWatch Alarm | `datacenter-alarm` |
| SNS Topic | `datacenter-sns-topic` |

### Task Completion Checklist
- [x] ✅ EC2 Instance `datacenter-ec2` provisioned with Ubuntu AMI.
- [x] ✅ CloudWatch Alarm `datacenter-alarm` created with >= 90% threshold.
- [x] ✅ Alarm configured for 5-minute (300s) average monitoring.
- [x] ✅ Alarm action linked to the required SNS topic.
- [x] ✅ Outputs for instance name and alarm name verified.
- [x] ✅ Logic centralized in `main.tf`.
- [x] ✅ `terraform validate` and `fmt` passed.
- [x] ✅ Final `terraform plan` returns **"No changes"**.
