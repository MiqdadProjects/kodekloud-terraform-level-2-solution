# 🌟 Task 12 - Provision SNS Topic, SQS Queue, and Subscription Using Terraform

## 📌 Task Description

The **Nautilus DevOps team** is implementing a **messaging system** in AWS using Amazon SNS and SQS. The goal is to create a **fan-out messaging pipeline** where an SNS topic acts as the central publisher and an SQS queue acts as a subscriber — ensuring messages published to the topic are durably delivered to the queue for asynchronous processing.

**Requirements:**
- Create an **SNS topic** named **`datacenter-sns-topic`**
- Create an **SQS queue** named **`datacenter-sqs-queue`**
- **Subscribe the SQS queue** to the SNS topic using an SNS subscription
- Set an **SQS queue policy** allowing SNS to send messages to the queue
- All resources must be defined in a single **`main.tf`** file
- Use **`outputs.tf`** with:
  - `kke_sns_topic_arn` — the ARN of the SNS topic
  - `kke_sqs_queue_url` — the URL of the SQS queue
- The Terraform working directory is **`/home/bob/terraform`**
- Verify that `terraform plan` returns **"No changes. Your infrastructure matches the configuration."**

👉 **Your task:** Build a reliable publish-subscribe messaging pipeline — SNS topic as publisher, SQS queue as subscriber — where messages are durably delivered and the queue is protected by a policy that only accepts messages from the authorized SNS topic.

💡 **Note:** Subscribing an SQS queue to an SNS topic **requires two steps**: (1) Creating the subscription (`aws_sns_topic_subscription`) and (2) Granting SNS permission to write to the queue via an `aws_sqs_queue_policy`. Without the queue policy, SNS cannot deliver messages even when a subscription exists.

---

## 🔧 Infrastructure Overview

**Target Environment:** AWS Cloud (us-east-1 region)  
**Provider:** AWS (Amazon Web Services)  
**Resources:**
- SNS Topic (`datacenter-sns-topic`) — message publisher
- SQS Queue (`datacenter-sqs-queue`) — message subscriber/buffer
- SQS Queue Policy — grants SNS permission to send messages to SQS
- SNS Topic Subscription — connects the SNS topic to the SQS queue

**Working Directory:** `/home/bob/terraform`

**Message Flow Architecture:**
```
Producer / Application
        │
        ▼  sns:Publish
aws_sns_topic (datacenter-sns-topic)
        │
        │  SNS delivers message to subscribed endpoints
        ▼
aws_sns_topic_subscription
  protocol = "sqs"
  endpoint = SQS queue ARN
        │
        │  sqs:SendMessage (allowed via queue policy)
        ▼
aws_sqs_queue (datacenter-sqs-queue)
        │
        ▼
Consumer / Worker processes messages
```

**Resource Summary:**
| Resource | Terraform Type | Name |
|----------|---------------|------|
| SNS Topic | `aws_sns_topic` | `datacenter-sns-topic` |
| SQS Queue | `aws_sqs_queue` | `datacenter-sqs-queue` |
| Queue Policy | `aws_sqs_queue_policy` | — (attached to queue) |
| SNS Subscription | `aws_sns_topic_subscription` | `sqs` protocol |

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **`aws_sns_topic`:** Creates the SNS publisher topic
- **`aws_sqs_queue`:** Creates the SQS subscriber queue with default settings
- **`aws_sqs_queue_policy`:** Grants the SNS topic permission to call `sqs:SendMessage` on the queue via a resource-based policy with `aws:SourceArn` condition for security
- **`aws_sns_topic_subscription`:** Subscribes the SQS queue to the SNS topic; `depends_on` ensures the queue policy is applied **before** the subscription is created
- **Implicit Dependencies:** Queue policy uses queue `id` and topic `arn`; subscription uses topic `arn` and queue `arn`

### 📁 File Structure
```bash
/home/bob/terraform/
├── main.tf      # SNS topic + SQS queue + queue policy + subscription
└── outputs.tf   # Output: SNS topic ARN + SQS queue URL
```

---

## 🚀 Implementation Steps

### Step 1: Navigate to Working Directory

```bash
cd /home/bob/terraform
```

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

Create `main.tf` with all four resources:

```bash
cat > main.tf << 'EOF'
# ---------------------------
# SNS Topic
# ---------------------------
resource "aws_sns_topic" "datacenter_topic" {
  name = "datacenter-sns-topic"
}

# ---------------------------
# SQS Queue
# ---------------------------
resource "aws_sqs_queue" "datacenter_queue" {
  name = "datacenter-sqs-queue"
}

# ---------------------------
# SQS Queue Policy (Allow SNS to send messages)
# ---------------------------
resource "aws_sqs_queue_policy" "queue_policy" {
  queue_url = aws_sqs_queue.datacenter_queue.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = "*"
        Action    = "sqs:SendMessage"
        Resource  = aws_sqs_queue.datacenter_queue.arn
        Condition = {
          ArnEquals = {
            "aws:SourceArn" = aws_sns_topic.datacenter_topic.arn
          }
        }
      }
    ]
  })
}

# ---------------------------
# SNS Subscription
# ---------------------------
resource "aws_sns_topic_subscription" "sqs_subscription" {
  topic_arn = aws_sns_topic.datacenter_topic.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.datacenter_queue.arn

  depends_on = [aws_sqs_queue_policy.queue_policy]
}
EOF
```

**Purpose:** Create the full SNS-to-SQS messaging pipeline — topic, queue, queue access policy, and the subscription that ties them together.

**Configuration Breakdown — Resource by Resource:**

**Resource 1 — SNS Topic (`aws_sns_topic.datacenter_topic`):**
- `name = "datacenter-sns-topic"` — Name of the SNS topic as required
- SNS topics are globally reachable by their ARN and can fan out to multiple endpoints (SQS, Lambda, HTTP, email, etc.)

**Resource 2 — SQS Queue (`aws_sqs_queue.datacenter_queue`):**
- `name = "datacenter-sqs-queue"` — Name of the SQS queue as required
- Uses all default settings: standard queue, 30-second visibility timeout, 4-day message retention
- The queue's `.id` attribute is its **URL** (used by the queue policy and output)
- The queue's `.arn` attribute is its **ARN** (used by the subscription endpoint)

**Resource 3 — SQS Queue Policy (`aws_sqs_queue_policy.queue_policy`):**
- `queue_url = aws_sqs_queue.datacenter_queue.id` — Targets the specific SQS queue
- `policy` — Resource-based policy granting SNS permission to send messages:
  - `Principal = "*"` — Required for SNS delivery (the `Condition` limits the actual source)
  - `Action = "sqs:SendMessage"` — Only message delivery is permitted; no other SQS operations
  - `Resource = aws_sqs_queue.datacenter_queue.arn` — Restricts to this specific queue
  - `Condition.ArnEquals."aws:SourceArn"` — **Security constraint**: only the specific SNS topic can trigger this permission; prevents other SNS topics from delivering to this queue

**Resource 4 — SNS Subscription (`aws_sns_topic_subscription.sqs_subscription`):**
- `topic_arn = aws_sns_topic.datacenter_topic.arn` — The publisher topic
- `protocol = "sqs"` — Delivery protocol for SQS endpoints
- `endpoint = aws_sqs_queue.datacenter_queue.arn` — The SQS queue's ARN as the delivery target
- `depends_on = [aws_sqs_queue_policy.queue_policy]` — **Critical:** ensures the queue policy is applied before SNS attempts to subscribe; without this, SNS may fail to validate the subscription if the queue doesn't yet have the `sqs:SendMessage` permission

**File Content (`main.tf`):**
```hcl
# ---------------------------
# SNS Topic
# ---------------------------
resource "aws_sns_topic" "datacenter_topic" {
  name = "datacenter-sns-topic"
}

# ---------------------------
# SQS Queue
# ---------------------------
resource "aws_sqs_queue" "datacenter_queue" {
  name = "datacenter-sqs-queue"
}

# ---------------------------
# SQS Queue Policy (Allow SNS to send messages)
# ---------------------------
resource "aws_sqs_queue_policy" "queue_policy" {
  queue_url = aws_sqs_queue.datacenter_queue.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = "*"
        Action    = "sqs:SendMessage"
        Resource  = aws_sqs_queue.datacenter_queue.arn
        Condition = {
          ArnEquals = {
            "aws:SourceArn" = aws_sns_topic.datacenter_topic.arn
          }
        }
      }
    ]
  })
}

# ---------------------------
# SNS Subscription
# ---------------------------
resource "aws_sns_topic_subscription" "sqs_subscription" {
  topic_arn = aws_sns_topic.datacenter_topic.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.datacenter_queue.arn

  depends_on = [aws_sqs_queue_policy.queue_policy]
}
```

---

### Step 3: Create Outputs Configuration File

Create `outputs.tf` to export SNS topic ARN and SQS queue URL:

```bash
cat > outputs.tf << 'EOF'
output "kke_sns_topic_arn" {
  value = aws_sns_topic.datacenter_topic.arn
}

output "kke_sqs_queue_url" {
  value = aws_sqs_queue.datacenter_queue.id
}
EOF
```

**Configuration Breakdown:**
- `kke_sns_topic_arn` → `aws_sns_topic.datacenter_topic.arn` → returns `arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic`
- `kke_sqs_queue_url` → `aws_sqs_queue.datacenter_queue.id` → returns the queue **URL** (the `.id` attribute of `aws_sqs_queue` is its URL, not ARN)

> 📝 **Key distinction:** For `aws_sqs_queue`:
> - `.id` = queue **URL** (e.g., `https://sqs.us-east-1.amazonaws.com/123.../datacenter-sqs-queue`)
> - `.arn` = queue **ARN** (e.g., `arn:aws:sqs:us-east-1:123...:datacenter-sqs-queue`)

**File Content (`outputs.tf`):**
```hcl
output "kke_sns_topic_arn" {
  value = aws_sns_topic.datacenter_topic.arn
}

output "kke_sqs_queue_url" {
  value = aws_sqs_queue.datacenter_queue.id
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
drwxr-xr-x 2 bob bob 4096 Feb 25 15:00 .
drwxr-xr-x 5 bob bob 4096 Feb 25 14:55 ..
-rw-r--r-- 1 bob bob  912 Feb 25 15:00 main.tf
-rw-r--r-- 1 bob bob  178 Feb 25 15:00 outputs.tf
```

---

### Step 5: Initialize Terraform

```bash
terraform init
```

**Expected Output:**
```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.31.0...
- Installed hashicorp/aws v5.31.0 (signed by HashiCorp)

Terraform has been successfully initialized!
```

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
- ✅ `aws_sns_topic`, `aws_sqs_queue`, `aws_sqs_queue_policy`, `aws_sns_topic_subscription` blocks are correct
- ✅ `jsonencode()` policy is valid HCL-to-JSON
- ✅ `depends_on` reference resolves correctly
- ✅ `protocol = "sqs"` is a valid SNS subscription type

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

**Expected Output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_sns_topic.datacenter_topic will be created
  + resource "aws_sns_topic" "datacenter_topic" {
      + arn  = (known after apply)
      + id   = (known after apply)
      + name = "datacenter-sns-topic"
    }

  # aws_sqs_queue.datacenter_queue will be created
  + resource "aws_sqs_queue" "datacenter_queue" {
      + arn  = (known after apply)
      + id   = (known after apply)
      + name = "datacenter-sqs-queue"
    }

  # aws_sqs_queue_policy.queue_policy will be created
  + resource "aws_sqs_queue_policy" "queue_policy" {
      + id        = (known after apply)
      + policy    = (known after apply)
      + queue_url = (known after apply)
    }

  # aws_sns_topic_subscription.sqs_subscription will be created
  + resource "aws_sns_topic_subscription" "sqs_subscription" {
      + arn       = (known after apply)
      + endpoint  = (known after apply)
      + id        = (known after apply)
      + protocol  = "sqs"
      + topic_arn = (known after apply)
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + kke_sns_topic_arn  = (known after apply)
  + kke_sqs_queue_url  = (known after apply)
```

**Plan Summary:**
- **Resources to Create:** 4 (topic + queue + queue policy + subscription)
- **Outputs:** Both known after apply (ARNs and URLs are assigned by AWS)

**Success Indicators:**
- ✅ SNS topic `datacenter-sns-topic` planned
- ✅ SQS queue `datacenter-sqs-queue` planned
- ✅ Queue policy and subscription planned
- ✅ Both outputs shown

---

### Step 9: Apply Configuration

```bash
terraform apply -auto-approve
```

**Expected Output:**
```
Plan: 4 to add, 0 to change, 0 to destroy.

aws_sns_topic.datacenter_topic: Creating...
aws_sns_topic.datacenter_topic: Creation complete after 1s [id=arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic]
aws_sqs_queue.datacenter_queue: Creating...
aws_sqs_queue.datacenter_queue: Creation complete after 1s [id=https://sqs.us-east-1.amazonaws.com/123456789012/datacenter-sqs-queue]
aws_sqs_queue_policy.queue_policy: Creating...
aws_sqs_queue_policy.queue_policy: Creation complete after 0s [id=https://sqs.us-east-1.amazonaws.com/123456789012/datacenter-sqs-queue]
aws_sns_topic_subscription.sqs_subscription: Creating...
aws_sns_topic_subscription.sqs_subscription: Creation complete after 0s [id=arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic:abc123-uuid]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

kke_sns_topic_arn  = "arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic"
kke_sqs_queue_url  = "https://sqs.us-east-1.amazonaws.com/123456789012/datacenter-sqs-queue"
```

**Creation Order (enforced by implicit + explicit dependencies):**
| Order | Resource | Reason |
|-------|----------|--------|
| 1st | `aws_sns_topic` + `aws_sqs_queue` | No dependencies (parallel) |
| 2nd | `aws_sqs_queue_policy` | Needs queue ID and SNS topic ARN |
| 3rd | `aws_sns_topic_subscription` | `depends_on` queue policy |

**Success Indicators:**
- ✅ SNS topic and SQS queue created in parallel (no dependency between them)
- ✅ Queue policy applied before subscription
- ✅ Subscription created last with SQS protocol
- ✅ `4 added, 0 changed, 0 destroyed`
- ✅ Both outputs display real ARN and URL values

---

### Step 10: Verify Final State (Critical Verification)

```bash
terraform plan
```

**Expected Output:**
```
aws_sns_topic.datacenter_topic: Refreshing state... [id=arn:aws:sns:...]
aws_sqs_queue.datacenter_queue: Refreshing state... [id=https://sqs...]
aws_sqs_queue_policy.queue_policy: Refreshing state...
aws_sns_topic_subscription.sqs_subscription: Refreshing state...

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```

**⚠️ Required before task submission.**

---

## ✅ Verification Steps

### Step 1: Verify All Resources in State

```bash
terraform state list
```

**Expected Output:**
```
aws_sns_topic.datacenter_topic
aws_sns_topic_subscription.sqs_subscription
aws_sqs_queue.datacenter_queue
aws_sqs_queue_policy.queue_policy
```

---

### Step 2: Inspect SNS Topic

```bash
terraform state show aws_sns_topic.datacenter_topic
```

**Expected Output:**
```
# aws_sns_topic.datacenter_topic:
resource "aws_sns_topic" "datacenter_topic" {
    arn  = "arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic"
    id   = "arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic"
    name = "datacenter-sns-topic"
}
```

---

### Step 3: Inspect SNS Subscription

```bash
terraform state show aws_sns_topic_subscription.sqs_subscription
```

**Expected Output:**
```
# aws_sns_topic_subscription.sqs_subscription:
resource "aws_sns_topic_subscription" "sqs_subscription" {
    endpoint  = "arn:aws:sqs:us-east-1:123456789012:datacenter-sqs-queue"
    id        = "arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic:abc123"
    protocol  = "sqs"
    topic_arn = "arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic"
}
```

**Verification Points:**
- ✅ `protocol = "sqs"`
- ✅ `endpoint` is the SQS queue ARN
- ✅ `topic_arn` matches the SNS topic

---

### Step 4: View Output Values

```bash
terraform output
```

**Expected Output:**
```
kke_sns_topic_arn  = "arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic"
kke_sqs_queue_url  = "https://sqs.us-east-1.amazonaws.com/123456789012/datacenter-sqs-queue"
```

---

### Step 5: End-to-End Message Test (Optional)

```bash
SNS_ARN=$(terraform output -raw kke_sns_topic_arn)
SQS_URL=$(terraform output -raw kke_sqs_queue_url)

# Publish a test message to the SNS topic
aws sns publish \
  --topic-arn "$SNS_ARN" \
  --message "Hello from SNS!"

# Receive the delivered message from SQS
aws sqs receive-message \
  --queue-url "$SQS_URL" \
  --query "Messages[0].Body" \
  --output text
```

**Expected Output (message body received from SQS):**
```
{"Type":"Notification","MessageId":"...","TopicArn":"arn:aws:sns:...","Message":"Hello from SNS!",...}
```

✅ This confirms the full SNS → SQS delivery pipeline is working.

---

## 🔍 Code Analysis

### Four-Resource Dependency Graph

```
aws_sns_topic.datacenter_topic     aws_sqs_queue.datacenter_queue
        │ .arn                              │ .id (URL)   │ .arn
        │                                  ▼             │
        │                       aws_sqs_queue_policy   ←─┘
        │                         (grants sqs:SendMessage)
        │                                  │
        │ .arn                             │  depends_on
        ▼                                  ▼
       aws_sns_topic_subscription.sqs_subscription
         topic_arn = SNS topic ARN
         protocol  = "sqs"
         endpoint  = SQS queue ARN
```

### Why the Queue Policy Uses `Principal = "*"` with a Condition

```hcl
Principal = "*"    # Allows any principal...
Condition = {
  ArnEquals = {
    "aws:SourceArn" = aws_sns_topic.datacenter_topic.arn
  }
}
# ...BUT ONLY when the request originates from this specific SNS topic
```

Using `Principal = "sns.amazonaws.com"` might seem more restrictive, but for SQS-SNS integration, `Principal = "*"` with the `aws:SourceArn` condition is the **AWS-recommended pattern**. It allows SNS (as a service) to deliver messages while restricting which specific topic can do so.

### `aws_sqs_queue` — `.id` vs `.arn`

| Attribute | Value Format | Used For |
|-----------|-------------|---------|
| `.id` | `https://sqs.region.amazonaws.com/account/queue-name` | Queue **URL** — operations (send, receive, delete) and policies |
| `.arn` | `arn:aws:sqs:region:account:queue-name` | **ARN** — subscription endpoints, IAM policies, cross-account access |

In this task:
- `kke_sqs_queue_url` output uses `.id` (the URL) ✅
- SNS subscription `endpoint` uses `.arn` ✅

### Why `depends_on` on the Subscription Is Critical

```
Without depends_on:              With depends_on [queue_policy]:
─────────────────────            ──────────────────────────────
1. Subscription created          1. Queue created
2. SNS tries to confirm          2. Queue policy applied
   subscription                  3. Subscription created
3. SQS rejects (no              4. SNS confirms subscription
   sqs:SendMessage               5. Pipeline fully operational ✅
   permission yet)
4. Subscription stuck
   in "PendingConfirmation"
```

---

## 🧪 Testing & Validation

### Complete Validation Workflow

```bash
# 1. Validate all files
terraform validate

# 2. Preview plan
terraform plan

# 3. Apply
terraform apply -auto-approve

# 4. Verify 4 resources in state
terraform state list

# 5. Check SNS topic name
terraform state show aws_sns_topic.datacenter_topic

# 6. Check subscription protocol and endpoint
terraform state show aws_sns_topic_subscription.sqs_subscription

# 7. View outputs
terraform output

# 8. Optional: end-to-end message test
aws sns publish --topic-arn "$(terraform output -raw kke_sns_topic_arn)" --message "test"
aws sqs receive-message --queue-url "$(terraform output -raw kke_sqs_queue_url)"

# 9. Final verification — must show "No changes"
terraform plan
```

### Key Validation Checklist

| Check | Expected Value | How to Verify |
|-------|---------------|---------------|
| SNS topic name | `datacenter-sns-topic` | `terraform state show aws_sns_topic.datacenter_topic` |
| SQS queue name | `datacenter-sqs-queue` | `terraform state show aws_sqs_queue.datacenter_queue` |
| Subscription protocol | `sqs` | `terraform state show aws_sns_topic_subscription.sqs_subscription` |
| Subscription endpoint | SQS queue ARN | `terraform state show aws_sns_topic_subscription.sqs_subscription` |
| SNS topic ARN output | `arn:aws:sns:...` | `terraform output kke_sns_topic_arn` |
| SQS URL output | `https://sqs...` | `terraform output kke_sqs_queue_url` |
| Final plan | `No changes` | `terraform plan` |

---

## 🛠️ Troubleshooting

### Issue 1: Subscription Stuck in `PendingConfirmation`

**Symptom:** `terraform state show aws_sns_topic_subscription.sqs_subscription` shows `pending_confirmation = true`.

**Cause:** The queue policy was not applied before the subscription was created, so SNS could not confirm the subscription.

**Solution:**
```bash
# Ensure depends_on is present in the subscription resource
# Then destroy and re-apply
terraform destroy -auto-approve
terraform apply -auto-approve
```

---

### Issue 2: Messages Not Delivered to SQS

**Symptom:** SNS publish succeeds, but no messages appear in the SQS queue.

**Cause:** The `aws_sqs_queue_policy` is missing or has an incorrect `aws:SourceArn` condition.

**Solution:**
```bash
# Verify queue policy is applied correctly
aws sqs get-queue-attributes \
  --queue-url "$(terraform output -raw kke_sqs_queue_url)" \
  --attribute-names Policy \
  --query "Attributes.Policy" \
  --output text | python3 -m json.tool
```

Confirm the policy contains:
- `Action: sqs:SendMessage`
- `Condition.ArnEquals."aws:SourceArn"` = SNS topic ARN

---

### Issue 3: SNS Topic or SQS Queue Name Already Exists

**Error:**
```
Error: creating SNS Topic: InvalidParameter: Topic already exists
```

**Solution:**
```bash
# Import existing topic into Terraform state
terraform import aws_sns_topic.datacenter_topic arn:aws:sns:us-east-1:123456789012:datacenter-sns-topic

# Import existing SQS queue
terraform import aws_sqs_queue.datacenter_queue https://sqs.us-east-1.amazonaws.com/123456789012/datacenter-sqs-queue

# Apply remaining resources (policy + subscription)
terraform apply -auto-approve
```

---

### Issue 4: Plan Shows Policy Changes After Apply

**Symptom:**
```
~ aws_sqs_queue_policy.queue_policy
  ~ policy = "{old JSON}" → "{new JSON}"
```

**Cause:** JSON formatting differences — Terraform may reorder keys or spacing in the rendered policy.

**Solution:** Use `terraform show` to inspect the stored policy and ensure both the stored and configured policies are semantically identical. If the only difference is key ordering, `terraform apply` should converge to "No changes" after the next apply.

---

### Issue 5: Cross-Account SNS-to-SQS Delivery Fails

**Symptom:** SNS topic in Account A cannot deliver to SQS queue in Account B.

**Cause:** Cross-account delivery requires the queue policy to allow the SNS topic's specific ARN from the other account.

**Solution:** Update the queue policy `Principal` to the specific SNS service ARN:
```hcl
Principal = {
  Service = "sns.amazonaws.com"
}
```
And ensure the `aws:SourceArn` condition includes the full cross-account topic ARN.

---

## 📚 Best Practices

### 1. Always Create the Queue Policy Before Subscribing
The `depends_on = [aws_sqs_queue_policy.queue_policy]` on the subscription is not optional — it prevents `PendingConfirmation` states that require manual intervention. Always enforce this ordering.

### 2. Use `aws:SourceArn` Condition for Security
The queue policy uses `Principal = "*"` (required for SNS delivery) but restricts it with `aws:SourceArn`. This **prevents confused deputy attacks** where another SNS topic tricks the queue into accepting unauthorized messages.

### 3. Use `.id` for SQS Queue URL and `.arn` for ARN References
These are different attributes in Terraform — mixing them up causes subtle errors:
```hcl
queue_url = aws_sqs_queue.queue.id    # ✅ URL for API operations
endpoint  = aws_sqs_queue.queue.arn   # ✅ ARN for SNS subscription
```

### 4. Enable Dead-Letter Queues (DLQ) for Production
For production, attach a DLQ to catch messages that fail delivery:
```hcl
resource "aws_sqs_queue" "datacenter_dlq" {
  name = "datacenter-sqs-dlq"
}

resource "aws_sqs_queue" "datacenter_queue" {
  name = "datacenter-sqs-queue"
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.datacenter_dlq.arn
    maxReceiveCount     = 5
  })
}
```

### 5. Enable SNS Message Filtering for Fan-Out Architectures
When multiple SQS queues subscribe to the same SNS topic, use subscription filter policies to route messages selectively:
```hcl
resource "aws_sns_topic_subscription" "sqs_subscription" {
  # ...
  filter_policy = jsonencode({ "eventType" = ["order_placed"] })
}
```

---

## 🎯 Task Completion Summary

### What Was Accomplished

| Resource | Name | Role |
|----------|------|------|
| SNS Topic | `datacenter-sns-topic` | Message publisher |
| SQS Queue | `datacenter-sqs-queue` | Message subscriber/buffer |
| Queue Policy | — | Grants SNS `sqs:SendMessage` permission |
| SNS Subscription | `sqs` protocol | Links topic → queue |

### Files Created

| File | Purpose |
|------|---------|
| `main.tf` | SNS topic + SQS queue + queue policy + subscription |
| `outputs.tf` | Exports `kke_sns_topic_arn` and `kke_sqs_queue_url` |

### Task Completion Checklist

- [x] ✅ SNS topic `datacenter-sns-topic` created
- [x] ✅ SQS queue `datacenter-sqs-queue` created
- [x] ✅ SQS queue policy grants `sqs:SendMessage` to the SNS topic (with `aws:SourceArn` condition)
- [x] ✅ SNS subscription created — SQS queue subscribed to SNS topic with `protocol = "sqs"`
- [x] ✅ `depends_on` ensures queue policy is applied before subscription
- [x] ✅ All resources defined in **single `main.tf`** file
- [x] ✅ `outputs.tf` exports `kke_sns_topic_arn` (SNS topic ARN)
- [x] ✅ `outputs.tf` exports `kke_sqs_queue_url` (SQS queue URL via `.id`)
- [x] ✅ `terraform validate` — **Success! The configuration is valid.**
- [x] ✅ `terraform fmt` — All files formatted
- [x] ✅ `terraform apply -auto-approve` — **4 resources added**
- [x] ✅ Final `terraform plan` — **"No changes. Your infrastructure matches the configuration."**

---

> 💡 **Key Takeaway:** This task implements the classic **SNS fan-out to SQS** pattern — the foundation of event-driven architectures in AWS. The **four-resource chain** (SNS topic → SQS queue → queue policy → SNS subscription) must be created in order. The critical design decision is using `depends_on` to guarantee the queue policy is in place before the subscription is created, preventing `PendingConfirmation` states. The `aws:SourceArn` condition in the queue policy enforces **least-privilege** even when `Principal = "*"` is required, ensuring only the specific SNS topic can deliver messages to the queue.
