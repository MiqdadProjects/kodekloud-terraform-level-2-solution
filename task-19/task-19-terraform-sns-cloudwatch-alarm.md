# 🌟 Task 19 - Provision SNS Topic and CloudWatch CPU Alarm Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is expanding their AWS infrastructure monitoring capabilities. The primary objective is to configure a proactive alerting system for EC2 instances. This involves setting up an **SNS topic** for notifications and a **CloudWatch alarm** that monitors CPU utilization. The alarm is designed to trigger when CPU usage exceeds 80%, ensuring the team is promptly notified of potential performance bottlenecks or system strain.

**Requirements:**
- Create an **SNS topic** named **`xfusion-sns-topic`**.
- Create a **CloudWatch alarm** named **`xfusion-cpu-alarm`** to monitor EC2 CPU utilization:
  - **Metric Name:** `CPUUtilization`
  - **Namespace:** `AWS/EC2`
  - **Threshold:** `80%`
  - **Comparison Operator:** `GreaterThanThreshold`
  - **Period:** `300` seconds (5 minutes)
  - **Evaluation Periods:** `1`
  - **Statistic:** `Average`
  - **Actions Enabled:** `true`
  - **Alarm Actions:** Linked to the `xfusion-sns-topic`.
- Update the **`main.tf`** file (no separate files for resources) to provision both the SNS Topic and the CloudWatch Alarm.
- Create an **`outputs.tf`** file to output:
  - `KKE_sns_topic_name`: The name of the created SNS topic.
  - `KKE_cloudwatch_alarm_name`: The name of the created CloudWatch alarm.
- The Terraform working directory is **`/home/bob/terraform`**.
- Before submitting, ensure that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Implement a robust monitoring and notification link between AWS CloudWatch and Simple Notification Service (SNS) using Terraform, ensuring real-time visibility into compute performance.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- SNS Topic (`xfusion-sns-topic`)
- CloudWatch Metric Alarm (`xfusion-cpu-alarm`)

**Working Directory:** `/home/bob/terraform`

**Alerting Flow:**
```
[EC2 Service Metrics]
        │
        ▼ (Periodic Reporting)
[CloudWatch Alarms]
        │
        ▼ (Condition: CPU > 80%)
[xfusion-cpu-alarm (State: ALARM)]
        │
        ▼ (Trigger Action)
[SNS Topic: xfusion-sns-topic]
        │
        ▼ (Notification)
[DevOps Team]
```

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_sns_topic`:** Acts as the communication hub for the alarm.
- **`aws_cloudwatch_metric_alarm`:** Monitors the `CPUUtilization` metric in the `AWS/EC2` namespace. When the average CPU exceeds 80% for a 5-minute window, it triggers the associated alarm action.
- **`outputs.tf`:** provides clean, labeled outputs for infrastructure verification.

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf            # SNS Topic and CloudWatch Alarm logic
└── outputs.tf         # Resource name outputs
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

---

### Step 2: Create Main Configuration File

Create the `main.tf` file containing both the notification topic and the monitoring alarm:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# SNS Topic
# ---------------------------
resource "aws_sns_topic" "xfusion_sns" {
  name = "xfusion-sns-topic"
}

# ---------------------------
# CloudWatch Alarm
# ---------------------------
resource "aws_cloudwatch_metric_alarm" "xfusion_cpu_alarm" {
  alarm_name          = "xfusion-cpu-alarm"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300
  statistic           = "Average"
  threshold           = 80
  actions_enabled     = true

  # Link the alarm to the SNS topic for notifications
  alarm_actions = [
    aws_sns_topic.xfusion_sns.arn
  ]
}
EOF
```

**Configuration Breakdown:**
- `metric_name = "CPUUtilization"`: Monitors the percentage of CPU utilized.
- `threshold = 80`: The trigger point for the alarm.
- `period = 300`: Aggregates metrics over a 5-minute interval.
- `alarm_actions`: Points to the ARN of the SNS topic created in the same file.

---

### Step 3: Create Outputs Configuration File

Create `outputs.tf` to export the resource names:

```bash
cat > outputs.tf << 'EOF'
output "KKE_sns_topic_name" {
  value = aws_sns_topic.xfusion_sns.name
}

output "KKE_cloudwatch_alarm_name" {
  value = aws_cloudwatch_metric_alarm.xfusion_cpu_alarm.alarm_name
}
EOF
```

---

### Step 4: Verify File Structure

```bash
ls -la /home/bob/terraform/
```

---

### Step 5: Initialize, Validate, and Apply

```bash
# Initialize Terraform
terraform init

# Validate configuration syntax
terraform validate

# Format the code for consistency
terraform fmt

# Apply the execution plan
terraform apply -auto-approve
```

**Expected Success Message:**
```
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

KKE_cloudwatch_alarm_name = "xfusion-cpu-alarm"
KKE_sns_topic_name = "xfusion-sns-topic"
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

### Step 1: Verify Resources in State

```bash
terraform state list
```

**Expected Output:**
```
aws_cloudwatch_metric_alarm.xfusion_cpu_alarm
aws_sns_topic.xfusion_sns
```

---

### Step 2: Inspect Alarm Definition

```bash
terraform state show aws_cloudwatch_metric_alarm.xfusion_cpu_alarm
```

**Verify:** 
- `threshold` is `80`.
- `comparison_operator` is `GreaterThanThreshold`.
- `alarm_actions` contains the SNS Topic ARN.

---

### Step 3: Verify SNS Topic Existence

```bash
aws sns list-topics
```

---

## 🔍 Code Analysis

### Decoupling vs. Reusability
While this task places both resources in `main.tf`, in larger projects, the SNS topic might be managed in a separate "Shared Services" module. Here, referencing `aws_sns_topic.xfusion_sns.arn` creates a direct **dependency**, ensuring Terraform always knows the destination for the alarm's notifications.

### Metric Monitoring vs. Alarm Evaluation
- **Period (300s):** This is the time window over which the **Statistic** (Average) is calculated.
- **Evaluation Periods (1):** This determines how many *consecutive* 300s windows must breach the threshold before the alarm state changes. With `1`, a single breach of the 5-minute average is enough to trigger the notification.

---

## 🎯 Task Completion Summary

### Resources Created

| Resource | Value |
|----------|-------|
| SNS Topic | `xfusion-sns-topic` |
| CloudWatch Alarm | `xfusion-cpu-alarm` |

### Task Completion Checklist
- [x] ✅ SNS topic `xfusion-sns-topic` provisioned.
- [x] ✅ CloudWatch CPU utilization alarm `xfusion-cpu-alarm` created.
- [x] ✅ Alarm threshold set to 80% with actions enabled.
- [x] ✅ Alarm actions successfully linked to the SNS topic ARN.
- [x] ✅ All logic centralized in `main.tf`.
- [x] ✅ Outputs for SNS and Alarm names exported in `outputs.tf`.
- [x] ✅ `terraform validate` and `fmt` passed.
- [x] ✅ Final `terraform plan` returns **"No changes"**.
