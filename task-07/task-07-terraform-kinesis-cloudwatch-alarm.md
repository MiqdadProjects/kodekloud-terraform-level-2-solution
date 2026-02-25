# 🌟 Task 07 - Provision a Kinesis Data Stream with CloudWatch Alarm Using Terraform

## 📌 Task Description

The **monitoring team** wants to improve **observability** into the streaming infrastructure. The goal is to implement an Amazon Kinesis Data Stream with shard-level metrics enabled, and configure a CloudWatch Alarm to trigger immediately if write throughput exceeds provisioned limits — ensuring proactive alerting before downstream consumers are impacted.

**Requirements:**
- Create a **Kinesis Data Stream** named **`xfusion-kinesis-stream`** with a shard count of **1**
- Enable **shard-level metrics** to track ingestion and throughput errors
- Create a **CloudWatch Alarm** named **`xfusion-kinesis-alarm`** monitoring the `WriteProvisionedThroughputExceeded` metric
- The alarm must trigger if the metric **exceeds a threshold of 1**
- All resources must be defined in a single **`main.tf`** file
- Use an **`outputs.tf`** file with:
  - `kke_kinesis_stream_name` — name of the Kinesis stream
  - `kke_kinesis_alarm_name` — name of the CloudWatch alarm
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Build a production-ready streaming observability stack — Kinesis Data Stream with granular shard metrics and a CloudWatch Alarm for real-time write throughput alerting.

💡 **Note:** `WriteProvisionedThroughputExceeded` is a critical Kinesis metric that counts the number of records rejected due to shard write limit exhaustion. Alarming on this metric with a threshold of `1` means **any single rejected write triggers the alarm** — the most sensitive alerting posture.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- Amazon Kinesis Data Stream (`xfusion-kinesis-stream`) — 1 shard, 24-hour retention, shard-level metrics enabled
- Amazon CloudWatch Metric Alarm (`xfusion-kinesis-alarm`) — monitors write throughput threshold violations

**Working Directory:** `/home/bob/terraform`

**Resource Configuration:**
| Resource | Name | Key Settings |
|----------|------|-------------|
| Kinesis Stream | `xfusion-kinesis-stream` | 1 shard, 24h retention, 4 shard metrics |
| CloudWatch Alarm | `xfusion-kinesis-alarm` | `WriteProvisionedThroughputExceeded > 1`, 60s period |

**Shard-Level Metrics Enabled:**
| Metric | Purpose |
|--------|---------|
| `IncomingBytes` | Total bytes written to the stream per second |
| `IncomingRecords` | Total records written per second |
| `WriteProvisionedThroughputExceeded` | Records rejected due to write limit — **alarmed on** |
| `ReadProvisionedThroughputExceeded` | Records rejected due to read limit |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Kinesis Data Stream (`aws_kinesis_stream`):** Single-shard streaming pipeline with shard-level metrics providing granular observability into ingestion health
- **CloudWatch Metric Alarm (`aws_cloudwatch_metric_alarm`):** Monitors `WriteProvisionedThroughputExceeded` for the stream; triggers when any write is throttled
- **Implicit Dependency:** The alarm's `dimensions` block references `aws_kinesis_stream.xfusion_stream.name`, ensuring the stream is created before the alarm

### 🎯 Implementation Strategy
1. Create `main.tf` with Kinesis stream (shard metrics) and CloudWatch alarm
2. Create `outputs.tf` to export stream name and alarm name
3. Initialize, validate, format, plan, apply, and verify

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf      # Kinesis stream + CloudWatch alarm
└── outputs.tf   # Output: stream name + alarm name
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

**Purpose:** Ensure all Terraform commands run in the correct directory.

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

Create `main.tf` with the Kinesis stream and CloudWatch alarm:

```bash
cat > main.tf << 'EOF'
resource "aws_kinesis_stream" "xfusion_stream" {
  name             = "xfusion-kinesis-stream"
  shard_count      = 1
  retention_period = 24

  shard_level_metrics = [
    "IncomingBytes",
    "IncomingRecords",
    "WriteProvisionedThroughputExceeded",
    "ReadProvisionedThroughputExceeded"
  ]
}

resource "aws_cloudwatch_metric_alarm" "xfusion_alarm" {
  alarm_name          = "xfusion-kinesis-alarm"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "WriteProvisionedThroughputExceeded"
  namespace           = "AWS/Kinesis"
  period              = 60
  statistic           = "Sum"
  threshold           = 1
  treat_missing_data  = "notBreaching"

  dimensions = {
    StreamName = aws_kinesis_stream.xfusion_stream.name
  }

  alarm_description = "Triggers when write throughput exceeds provisioned limit"
}
EOF
```

**Purpose:** Define both the Kinesis stream with observability metrics and the CloudWatch alarm that monitors write throttling.

**Configuration Breakdown:**

**Kinesis Stream Resource (`aws_kinesis_stream.xfusion_stream`):**
- `name = "xfusion-kinesis-stream"` — Stream name as required
- `shard_count = 1` — Single shard (1 MB/s write, 2 MB/s read capacity)
- `retention_period = 24` — Retains records for 24 hours (default minimum)
- `shard_level_metrics` — Enables per-shard CloudWatch metrics; without this, only stream-level metrics are available

**CloudWatch Alarm Resource (`aws_cloudwatch_metric_alarm.xfusion_alarm`):**
- `alarm_name = "xfusion-kinesis-alarm"` — Alarm name as required
- `comparison_operator = "GreaterThanThreshold"` — Alarm fires when metric value > threshold
- `evaluation_periods = 1` — One data point is enough to trigger the alarm (immediate alerting)
- `metric_name = "WriteProvisionedThroughputExceeded"` — Monitors write throttling events
- `namespace = "AWS/Kinesis"` — Kinesis CloudWatch metric namespace
- `period = 60` — Evaluates metric over a 60-second rolling window
- `statistic = "Sum"` — Sums all throttled write events in the period
- `threshold = 1` — Alarm triggers if **any** write is throttled (most sensitive setting)
- `treat_missing_data = "notBreaching"` — Missing data does not trigger the alarm (stream may have no traffic)
- `dimensions.StreamName` — Scopes the alarm to this specific Kinesis stream (implicit dependency on stream name)

**Alarm Logic Explained:**
```
IF Sum(WriteProvisionedThroughputExceeded) over 60 seconds > 1
THEN → ALARM state triggered
```

**File Content (`main.tf`):**
```hcl
resource "aws_kinesis_stream" "xfusion_stream" {
  name             = "xfusion-kinesis-stream"
  shard_count      = 1
  retention_period = 24

  shard_level_metrics = [
    "IncomingBytes",
    "IncomingRecords",
    "WriteProvisionedThroughputExceeded",
    "ReadProvisionedThroughputExceeded"
  ]
}

resource "aws_cloudwatch_metric_alarm" "xfusion_alarm" {
  alarm_name          = "xfusion-kinesis-alarm"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "WriteProvisionedThroughputExceeded"
  namespace           = "AWS/Kinesis"
  period              = 60
  statistic           = "Sum"
  threshold           = 1
  treat_missing_data  = "notBreaching"

  dimensions = {
    StreamName = aws_kinesis_stream.xfusion_stream.name
  }

  alarm_description = "Triggers when write throughput exceeds provisioned limit"
}
```

---

### Step 3: Create Outputs Configuration File

Create `outputs.tf` to export the stream name and alarm name:

```bash
cat > outputs.tf << 'EOF'
output "kke_kinesis_stream_name" {
  value = aws_kinesis_stream.xfusion_stream.name
}

output "kke_kinesis_alarm_name" {
  value = aws_cloudwatch_metric_alarm.xfusion_alarm.alarm_name
}
EOF
```

**Purpose:** Export stream and alarm names for verification and downstream use.

**Configuration Breakdown:**
- `kke_kinesis_stream_name` — Returns the Kinesis stream name: `"xfusion-kinesis-stream"`
- `kke_kinesis_alarm_name` — Returns the CloudWatch alarm name: `"xfusion-kinesis-alarm"`

**File Content (`outputs.tf`):**
```hcl
output "kke_kinesis_stream_name" {
  value = aws_kinesis_stream.xfusion_stream.name
}

output "kke_kinesis_alarm_name" {
  value = aws_cloudwatch_metric_alarm.xfusion_alarm.alarm_name
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
-rw-r--r-- 1 bob bob  598 Feb 25 11:00 main.tf
-rw-r--r-- 1 bob bob  178 Feb 25 11:00 outputs.tf
```

**Success Indicators:**
- ✅ Both files present: `main.tf` and `outputs.tf`
- ✅ No extra `.tf` files created

---

### Step 5: Initialize Terraform

```bash
terraform init
```

**Purpose:** Download the AWS provider and initialize the working directory.

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
- ✅ `.terraform/` and `.terraform.lock.hcl` created

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
- ✅ `aws_kinesis_stream` block attributes are correct
- ✅ `shard_level_metrics` list values are valid metric names
- ✅ `aws_cloudwatch_metric_alarm` block attributes are valid
- ✅ `dimensions` map correctly references the stream name

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

**Purpose:** Preview the creation of the Kinesis stream and CloudWatch alarm.

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_cloudwatch_metric_alarm.xfusion_alarm will be created
  + resource "aws_cloudwatch_metric_alarm" "xfusion_alarm" {
      + alarm_description   = "Triggers when write throughput exceeds provisioned limit"
      + alarm_name          = "xfusion-kinesis-alarm"
      + arn                 = (known after apply)
      + comparison_operator = "GreaterThanThreshold"
      + dimensions          = {
          + "StreamName" = "xfusion-kinesis-stream"
        }
      + evaluate_low_sample_count_percentiles = (known after apply)
      + evaluation_periods  = 1
      + id                  = (known after apply)
      + metric_name         = "WriteProvisionedThroughputExceeded"
      + namespace           = "AWS/Kinesis"
      + period              = 60
      + statistic           = "Sum"
      + threshold           = 1
      + treat_missing_data  = "notBreaching"
    }

  # aws_kinesis_stream.xfusion_stream will be created
  + resource "aws_kinesis_stream" "xfusion_stream" {
      + arn                   = (known after apply)
      + id                    = (known after apply)
      + name                  = "xfusion-kinesis-stream"
      + retention_period      = 24
      + shard_count           = 1
      + shard_level_metrics   = [
          + "IncomingBytes",
          + "IncomingRecords",
          + "ReadProvisionedThroughputExceeded",
          + "WriteProvisionedThroughputExceeded",
        ]
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + kke_kinesis_alarm_name  = "xfusion-kinesis-alarm"
  + kke_kinesis_stream_name = "xfusion-kinesis-stream"

───────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't
guarantee to take exactly these actions if you run "terraform apply" now.
```

**Plan Summary:**
- **Resources to Create:** 2 (Kinesis stream + CloudWatch alarm)
- **Resources to Change:** 0
- **Resources to Destroy:** 0
- **Outputs:** Both names known at plan time (no `(known after apply)`)

**Success Indicators:**
- ✅ Kinesis stream planned with correct name, shard count, and 4 metrics
- ✅ CloudWatch alarm planned with correct metric, threshold, and stream dimension
- ✅ Both output names pre-populated in the plan

---

### Step 9: Apply Configuration

```bash
terraform apply -auto-approve
```

**Purpose:** Provision the Kinesis stream and CloudWatch alarm in AWS.

**Expected Output:**
```
Plan: 2 to add, 0 to change, 0 to destroy.

aws_kinesis_stream.xfusion_stream: Creating...
aws_kinesis_stream.xfusion_stream: Still creating... [10s elapsed]
aws_kinesis_stream.xfusion_stream: Creation complete after 16s [id=arn:aws:kinesis:us-east-1:123456789012:stream/xfusion-kinesis-stream]
aws_cloudwatch_metric_alarm.xfusion_alarm: Creating...
aws_cloudwatch_metric_alarm.xfusion_alarm: Creation complete after 1s [id=xfusion-kinesis-alarm]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

kke_kinesis_alarm_name  = "xfusion-kinesis-alarm"
kke_kinesis_stream_name = "xfusion-kinesis-stream"
```

**Success Indicators:**
- ✅ Kinesis stream created first (CloudWatch alarm depends on stream name)
- ✅ CloudWatch alarm created after Kinesis stream is available
- ✅ Both output names displayed correctly
- ✅ `2 added, 0 changed, 0 destroyed`

---

### Step 10: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Expected Output:**
```
aws_kinesis_stream.xfusion_stream: Refreshing state... [id=arn:aws:kinesis:us-east-1:123456789012:stream/xfusion-kinesis-stream]
aws_cloudwatch_metric_alarm.xfusion_alarm: Refreshing state... [id=xfusion-kinesis-alarm]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**Success Indicators:**
- ✅ **Critical:** Message shows **"No changes. Your infrastructure matches the configuration."**
- ✅ Both resources refreshed from live AWS state
- ✅ No drift detected — task is ready for submission

---

## ✅ Verification Steps

### Step 1: Verify Terraform State

```bash
terraform state list
```

**Expected Output:**
```
aws_cloudwatch_metric_alarm.xfusion_alarm
aws_kinesis_stream.xfusion_stream
```

---

### Step 2: Inspect Kinesis Stream Details

```bash
terraform state show aws_kinesis_stream.xfusion_stream
```

**Expected Output:**
```
# aws_kinesis_stream.xfusion_stream:
resource "aws_kinesis_stream" "xfusion_stream" {
    arn              = "arn:aws:kinesis:us-east-1:123456789012:stream/xfusion-kinesis-stream"
    id               = "arn:aws:kinesis:us-east-1:123456789012:stream/xfusion-kinesis-stream"
    name             = "xfusion-kinesis-stream"
    retention_period = 24
    shard_count      = 1
    shard_level_metrics = [
        "IncomingBytes",
        "IncomingRecords",
        "ReadProvisionedThroughputExceeded",
        "WriteProvisionedThroughputExceeded",
    ]
}
```

**Verification Points:**
- ✅ `shard_count = 1`
- ✅ `retention_period = 24`
- ✅ All 4 `shard_level_metrics` enabled
- ✅ `name = "xfusion-kinesis-stream"`

---

### Step 3: Inspect CloudWatch Alarm Details

```bash
terraform state show aws_cloudwatch_metric_alarm.xfusion_alarm
```

**Expected Output:**
```
# aws_cloudwatch_metric_alarm.xfusion_alarm:
resource "aws_cloudwatch_metric_alarm" "xfusion_alarm" {
    alarm_description   = "Triggers when write throughput exceeds provisioned limit"
    alarm_name          = "xfusion-kinesis-alarm"
    comparison_operator = "GreaterThanThreshold"
    dimensions          = {
        "StreamName" = "xfusion-kinesis-stream"
    }
    evaluation_periods  = 1
    metric_name         = "WriteProvisionedThroughputExceeded"
    namespace           = "AWS/Kinesis"
    period              = 60
    statistic           = "Sum"
    threshold           = 1
    treat_missing_data  = "notBreaching"
}
```

**Verification Points:**
- ✅ `metric_name = "WriteProvisionedThroughputExceeded"`
- ✅ `threshold = 1`
- ✅ `evaluation_periods = 1` (immediate alerting)
- ✅ `dimensions.StreamName = "xfusion-kinesis-stream"` (scoped to correct stream)

---

### Step 4: View Output Values

```bash
terraform output
```

**Expected Output:**
```
kke_kinesis_alarm_name  = "xfusion-kinesis-alarm"
kke_kinesis_stream_name = "xfusion-kinesis-stream"
```

---

### Step 5: AWS CLI Verification (Optional)

```bash
# Verify Kinesis stream is active with shard metrics
aws kinesis describe-stream-summary \
  --stream-name xfusion-kinesis-stream \
  --query "StreamDescriptionSummary.{Name:StreamName,Status:StreamStatus,Shards:OpenShardCount}" \
  --output table

# Verify CloudWatch alarm is configured correctly
aws cloudwatch describe-alarms \
  --alarm-names xfusion-kinesis-alarm \
  --query "MetricAlarms[].{Name:AlarmName,Metric:MetricName,Threshold:Threshold,State:StateValue}" \
  --output table
```

---

## 🔍 Code Analysis

### Observability Architecture

```
Kinesis Data Stream: xfusion-kinesis-stream
  │  Shard 1 (1 MB/s write capacity)
  │  Shard-level metrics ENABLED ──────────────────────────────────┐
  │  ├── IncomingBytes                                             │
  │  ├── IncomingRecords                                           │
  │  ├── WriteProvisionedThroughputExceeded  ←── alarmed on        │
  │  └── ReadProvisionedThroughputExceeded                         │
  │                                                                │
  ▼                                                                ▼
Producers write records           CloudWatch Metrics (per-shard, per-minute)
  │                                           │
  │ If write limit (1 MB/s) exceeded          │
  └─── throttled record counted ──────────────▼
                                   xfusion-kinesis-alarm
                                   (WriteProvisionedThroughputExceeded > 1)
                                   → ALARM state triggered instantly
```

### CloudWatch Alarm Configuration Analysis

| Parameter | Value | Explanation |
|-----------|-------|-------------|
| `comparison_operator` | `GreaterThanThreshold` | Triggers when metric > threshold |
| `evaluation_periods` | `1` | Single data point triggers alarm (no waiting) |
| `period` | `60` | 60-second evaluation window |
| `statistic` | `Sum` | Count all throttled writes in the window |
| `threshold` | `1` | Any throttling event triggers the alarm |
| `treat_missing_data` | `notBreaching` | No data = OK (stream may be idle) |

### Shard-Level vs Stream-Level Metrics

| Metric Level | Default | Granularity | Cost |
|-------------|---------|-------------|------|
| **Stream-level** | ✅ Enabled | Per-stream aggregates only | Free |
| **Shard-level** | ❌ Disabled | Per-shard granularity | Additional CloudWatch cost |

Enabling `shard_level_metrics` in this task is essential because the CloudWatch alarm uses the `WriteProvisionedThroughputExceeded` metric at the shard level — without it, this metric would not be published.

### Implicit Dependency

```hcl
# aws_cloudwatch_metric_alarm.xfusion_alarm depends on stream name:
dimensions = {
  StreamName = aws_kinesis_stream.xfusion_stream.name
  #            ──────────────────────────────────────
  #            This attribute reference creates an implicit dependency.
  #            Kinesis stream is created BEFORE the alarm.
}
```

No explicit `depends_on` is needed — the `dimensions` reference handles it automatically.

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate configuration
terraform validate

# 2. Preview the plan (2 resources)
terraform plan

# 3. Apply
terraform apply -auto-approve

# 4. Confirm both resources in state
terraform state list

# 5. Confirm shard metrics on stream
terraform state show aws_kinesis_stream.xfusion_stream

# 6. Confirm alarm threshold and metric
terraform state show aws_cloudwatch_metric_alarm.xfusion_alarm

# 7. View outputs
terraform output

# 8. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| Stream name | `xfusion-kinesis-stream` | `terraform output kke_kinesis_stream_name` |
| Shard count | `1` | `terraform state show aws_kinesis_stream.xfusion_stream` |
| Shard metrics | All 4 enabled | `terraform state show aws_kinesis_stream.xfusion_stream` |
| Alarm name | `xfusion-kinesis-alarm` | `terraform output kke_kinesis_alarm_name` |
| Alarm metric | `WriteProvisionedThroughputExceeded` | `terraform state show aws_cloudwatch_metric_alarm.xfusion_alarm` |
| Alarm threshold | `1` | `terraform state show aws_cloudwatch_metric_alarm.xfusion_alarm` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: Kinesis Stream Name Already Exists

**Error:**
```
Error: creating Kinesis Stream: ResourceInUseException: Stream xfusion-kinesis-stream under account already exists
```

**Solution:**
```bash
# Delete the existing stream via AWS CLI
aws kinesis delete-stream --stream-name xfusion-kinesis-stream

# Wait for deletion to complete
aws kinesis describe-stream-summary --stream-name xfusion-kinesis-stream
# Should return ResourceNotFoundException when deleted

# Then re-apply
terraform apply -auto-approve
```

---

### Issue 2: Invalid Shard-Level Metric Name

**Error:**
```
Error: expected shard_level_metrics to be one of [IncomingBytes IncomingRecords IteratorAgeMilliseconds OutgoingBytes ...]
```

**Cause:** An invalid metric name was used in `shard_level_metrics`.

**Solution:** Use only these valid shard-level metric names:
```hcl
shard_level_metrics = [
  "IncomingBytes",
  "IncomingRecords",
  "WriteProvisionedThroughputExceeded",
  "ReadProvisionedThroughputExceeded",
  "IteratorAgeMilliseconds",
  "OutgoingBytes",
  "OutgoingRecords"
]
```

---

### Issue 3: CloudWatch Alarm Not Triggering

**Symptom:** Writes are being throttled but the alarm stays in `OK` state.

**Cause:** Shard-level metrics may not have been enabled before the alarm was created, so no data points exist.

**Solution:**
```bash
# Verify shard-level metrics are enabled
aws kinesis list-stream-consumers \
  --stream-arn $(aws kinesis describe-stream-summary \
    --stream-name xfusion-kinesis-stream \
    --query StreamDescriptionSummary.StreamARN --output text)

# Check if metric data exists in CloudWatch
aws cloudwatch get-metric-statistics \
  --namespace AWS/Kinesis \
  --metric-name WriteProvisionedThroughputExceeded \
  --dimensions Name=StreamName,Value=xfusion-kinesis-stream \
  --start-time $(date -u -d '1 hour ago' '+%Y-%m-%dT%H:%M:%SZ') \
  --end-time $(date -u '+%Y-%m-%dT%H:%M:%SZ') \
  --period 60 \
  --statistics Sum \
  --output table
```

---

### Issue 4: Alarm `treat_missing_data` Causing False Alarms

**Symptom:** Alarm enters `ALARM` state even when the stream has no traffic.

**Cause:** `treat_missing_data` is not set to `notBreaching`.

**Solution:** Ensure `main.tf` has:
```hcl
treat_missing_data = "notBreaching"
```
This prevents the alarm from triggering when there is no data (e.g., stream is idle or just created).

---

### Issue 5: Plan Shows Changes After Apply

**Symptom:**
```
Plan: 0 to add, 1 to change, 0 to destroy.
~ aws_kinesis_stream.xfusion_stream
  ~ retention_period = 24 -> 168
```

**Cause:** AWS may update the default `retention_period` or other stream attributes after creation.

**Solution:** Explicitly set all stream attributes in `main.tf` to match AWS defaults and re-apply:
```bash
terraform apply -auto-approve
terraform plan   # Should now show "No changes"
```

---

## 📚 Best Practices

### 1. Always Enable Shard-Level Metrics for Alarms
Stream-level metrics are averages across all shards. For per-shard throttling visibility and for CloudWatch alarms on `WriteProvisionedThroughputExceeded`, **shard-level metrics must be explicitly enabled**.

### 2. Set `treat_missing_data = "notBreaching"` for Streaming Workloads
Kinesis streams may be idle during off-peak hours. Missing data should not trigger a false alarm:
```hcl
treat_missing_data = "notBreaching"   # ✅ Idle stream = OK
treat_missing_data = "breaching"      # ❌ Idle stream = ALARM
```

### 3. Use `evaluation_periods = 1` for Immediate Alerting
For write throttling, a single data point is enough evidence. Multiple evaluation periods add unnecessary delay to critical alerts.

### 4. Scale Shards to Fix Sustained Throttling
If `WriteProvisionedThroughputExceeded` alarms frequently, the solution is to increase `shard_count`:
```hcl
# Each shard supports 1 MB/s or 1,000 records/s write throughput
shard_count = 2   # 2 MB/s write capacity
shard_count = 4   # 4 MB/s write capacity
```

### 5. Add SNS Notification Actions to Alarms (Production)
For production, connect the alarm to an SNS topic for email/pager notifications:
```hcl
resource "aws_cloudwatch_metric_alarm" "xfusion_alarm" {
  # ...
  alarm_actions = [aws_sns_topic.kinesis_alerts.arn]
  ok_actions    = [aws_sns_topic.kinesis_alerts.arn]
}
```

### 6. Tag Kinesis Streams for Cost Allocation
```hcl
resource "aws_kinesis_stream" "xfusion_stream" {
  # ...
  tags = {
    Name        = "xfusion-kinesis-stream"
    Environment = "production"
    Team        = "devops"
  }
}
```

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Name | Key Settings |
|----------|------|-------------|
| Kinesis Data Stream | `xfusion-kinesis-stream` | 1 shard, 4 shard metrics, 24h retention |
| CloudWatch Alarm | `xfusion-kinesis-alarm` | `WriteProvisionedThroughputExceeded > 1`, 60s window |

### Files Created

| File | Purpose |
|------|---------|
| `main.tf` | Kinesis stream + CloudWatch alarm resource definitions |
| `outputs.tf` | Exports `kke_kinesis_stream_name` and `kke_kinesis_alarm_name` |

### Task Completion Checklist

- [x] ✅ Kinesis stream `xfusion-kinesis-stream` created with `shard_count = 1`
- [x] ✅ Shard-level metrics enabled: `IncomingBytes`, `IncomingRecords`, `WriteProvisionedThroughputExceeded`, `ReadProvisionedThroughputExceeded`
- [x] ✅ CloudWatch alarm `xfusion-kinesis-alarm` created
- [x] ✅ Alarm monitors `WriteProvisionedThroughputExceeded` metric
- [x] ✅ Alarm triggers when metric exceeds threshold of **1**
- [x] ✅ All resources defined in **single `main.tf`** file
- [x] ✅ `outputs.tf` exports `kke_kinesis_stream_name`
- [x] ✅ `outputs.tf` exports `kke_kinesis_alarm_name`
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply -auto-approve` — **2 resources added**
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task implements a complete **streaming observability stack** using Terraform. The `shard_level_metrics` list on the Kinesis stream is a prerequisite for fine-grained CloudWatch alarming — without it, the `WriteProvisionedThroughputExceeded` metric would not be available at the shard level. The CloudWatch alarm with `threshold = 1` and `evaluation_periods = 1` ensures **zero-tolerance alerting** — the first throttled write immediately triggers the alarm, making this configuration ideal for latency-sensitive streaming pipelines where any data loss is unacceptable.
