# Lab 6: Monitoring, Logging & Governance

## Lab Overview

This **standalone lab** teaches you how to observe, troubleshoot, and govern workloads on Google Cloud Platform (GCP). You will learn three core pillars:

| Pillar | What It Does | Why It Matters |
|--------|--------------|----------------|
| **Monitoring** | Tracks metrics (CPU, memory, disk) and shows them on dashboards | Catch problems before users notice |
| **Logging** | Records events and messages from your applications and infrastructure | Understand *why* something failed |
| **Governance** | Labels, policies, and cost controls | Keep costs predictable and enforce security rules |

---

## Lab Objectives

By the end of this lab, you will:

- Use **Cloud Monitoring** to view VM and storage metrics
- Create a **custom dashboard** and **alert policy**
- Explore **Cloud Logging** for troubleshooting and auditing
- Create **logs-based metrics** to turn log events into alerts
- Apply **labels** for cost tracking and governance
- Understand **organization policies** for operational hardening

---

## Estimated Time

**90–120 minutes**

---

## Prerequisites (Standalone)

This lab is designed to run **on its own**. You need:

- A **Google Cloud account** (free trial or paid)
- **Owner** or **Editor** role on a GCP project
- A modern browser (Chrome recommended)

**If you don't have a VM yet:** Follow the "Quick Setup" section below to create one in about 5 minutes.

---

## Understanding the Concepts

### What is Cloud Monitoring?

Cloud Monitoring collects **metrics** (numbers over time) from your resources—VMs, storage, databases, etc. Think of it as a health monitor: it tells you if your system is running normally or under stress.

**Example:** CPU utilization at 95% for 5 minutes might mean your app is overloaded and could crash soon.

### What is Cloud Logging?

Cloud Logging stores **logs**—text records of what happened. Every time something occurs (a user logs in, a VM starts, an error occurs), a log entry is created. Logs help you answer: *"What exactly went wrong?"*

**Example:** A log entry might say: `"Connection refused to database at 10.23.45.67"`—now you know the root cause.

### What is Governance?

Governance means **controlling how resources are used** and **who pays for what**. In GCP, this includes:

- **Labels** – Tags on resources (e.g., `environment=prod`, `team=finance`) for cost allocation
- **Organization Policies** – Rules that restrict what can be created (e.g., "No public IPs in production")
- **Audit Logs** – Records of who did what, for compliance

---

## Lab Scenario: Real-World Context

You operate a **small web application** that serves customers 24/7. Your goals:

1. **Detect issues early** – Know when CPU or disk is high before the app goes down
2. **Get alerted** – Receive an email when something needs attention
3. **Investigate quickly** – Use logs to find the root cause of failures
4. **Control costs** – Use labels to see which team or project is spending what
5. **Stay compliant** – Have an audit trail of who changed what

This lab walks you through setting up exactly that.

---

## Quick Setup (If You Don't Have a VM)

**Skip this section if you already have a running VM.**

1. Open [Google Cloud Console](https://console.cloud.google.com)
2. Select or create a project
3. Go to **Compute Engine → VM Instances**
4. Click **Create Instance**
5. Use:
   - **Name:** `lab-vm`
   - **Region:** `us-central1` (or your preferred region)
   - **Machine type:** `e2-micro` (free tier eligible)
   - **Boot disk:** Default (10 GB)
6. Click **Create**
7. Wait 1–2 minutes for the VM to start

You now have a VM to monitor and log.

---

## Step 1: Verify Monitoring Is Enabled

**Goal:** Confirm that GCP is collecting metrics from your resources.

### Steps

1. In the Cloud Console, open the **navigation menu** (☰) and go to **Monitoring**
2. If prompted to set up a workspace, choose **Create workspace** and select your project
3. In the left menu, click **Metrics Explorer** (under "Monitoring")
4. You should see a chart area. Select:
   - **Resource type:** `VM Instance`
   - **Metric:** `CPU utilization`
5. If you have a VM, select it from the filter. You should see a graph of CPU usage.

### Expected Result

- Metrics Explorer opens without errors
- CPU utilization (or similar) is visible for your VM

### Learning Point

Monitoring is **enabled by default** for GCP resources. You don't need to install agents for basic VM metrics—GCP collects them automatically.

---

## Step 2: Explore VM Metrics

**Goal:** Understand what metrics are available and how to read them.

### Steps

1. Stay in **Monitoring → Metrics Explorer**
2. Configure:
   - **Resource type:** `VM Instance`
   - **Metric:** `CPU utilization`
3. In the filter, select your VM instance
4. Observe the graph—it shows CPU usage over time
5. Try changing the metric to:
   - `Disk read bytes`
   - `Network bytes received`

### Expected Result

- Live or recent data appears in the graph
- You can switch between different metrics

### Learning Point

Metrics are **time-series data**. Each point is a value at a specific time. Use them to spot trends (e.g., CPU slowly increasing) or sudden spikes.

---

## Step 3: Create a Custom Dashboard

**Goal:** Build a single view that shows the health of your VM at a glance.

### Steps

1. Go to **Monitoring → Dashboards**
2. Click **Create Dashboard**
3. Enter dashboard name: `infra-ops-dashboard`
4. Click **Add Widget**
5. Choose **Line chart**
6. Configure:
   - **Resource type:** `VM Instance`
   - **Metric:** `CPU utilization`
   - Select your VM in the filter
7. Click **Add Widget** again
8. Add a second widget:
   - **Metric:** `Network bytes received`
   - Same resource type and VM
9. Click **Save** (top right)

### Expected Result

- A dashboard with two charts: CPU and network traffic
- You can return to this dashboard anytime from **Monitoring → Dashboards**

### Learning Point

Dashboards give you **at-a-glance system health**. In production, teams use them to monitor dozens of resources on one screen.

---

## Step 4: Create an Alert Policy (CPU)

**Goal:** Get notified when CPU usage is too high for too long.

### Steps

1. Go to **Monitoring → Alerting**
2. Click **Create Policy**
3. Under **Conditions**, click **Add Condition**
4. Configure:
   - **Target:** Select your VM (or "Any VM" for all)
   - **Metric:** `CPU utilization`
   - **Condition:** `is above`
   - **Threshold:** `60` (percent)
   - **For:** `1 minute`
5. Click **Next**
6. Under **Notifications**, add your email:
   - Click **Notification channels**
   - Add **Email** and enter your address
   - Save
7. Select your email as the notification channel
8. Click **Next**
9. Name the policy: `high-cpu-alert`
10. Click **Create Policy**

### Expected Result

- A new alert policy appears in the list
- You will receive an email when CPU stays above 60% for 1 minute

### Learning Point

Alerts should be **actionable**. Too many alerts (e.g., threshold at 10%) cause "alert fatigue"—people ignore them. Set thresholds that mean "something needs attention."

---

## Step 5: Trigger the Alert (Optional)

**Goal:** See the alert in action by generating CPU load.

### Steps

1. Go to **Compute Engine → VM Instances**
2. Click **SSH** next to your VM to open a browser-based terminal
3. Run:

```bash
sudo apt-get update && sudo apt-get install -y stress
stress --cpu 2 --timeout 180
```

4. Leave the command running (it will stop after 3 minutes)
5. Return to **Monitoring → Dashboards** and watch CPU spike
6. Within a few minutes, check your email for the alert notification

### Expected Result

- CPU graph shows a sharp increase
- You receive an email: "Incident opened: high-cpu-alert"

### Learning Point

Testing alerts ensures they work when real problems occur. Schedule regular "alert drills" in production.

---

## Step 6: Explore Cloud Logging

**Goal:** Use logs to troubleshoot and understand what is happening on your VM.

### Steps

1. Go to **Logging → Logs Explorer** (from the navigation menu)
2. In the **Query** box, enter:

```
resource.type="gce_instance"
```

3. Click **Run query**
4. Browse the log entries. You should see:
   - **System logs** – OS-level events
   - **Startup logs** – What ran when the VM booted
   - **SSH logs** – When someone connected via SSH

### Expected Result

- A list of log entries from your VM
- Each entry shows timestamp, severity, and message

### Learning Point

Logs are the **first stop for troubleshooting**. When something fails, logs often contain the exact error message and context.

---

## Step 7: Review Audit Logs

**Goal:** See how GCP records every administrative action for compliance.

### Steps

1. Stay in **Logging → Logs Explorer**
2. Clear the query and enter:

```
logName:"cloudaudit.googleapis.com/activity"
```

   Or use the **Log name** dropdown and select **Cloud Audit Logs → Activity**.

3. Click **Run query**
4. Look for entries such as:
   - `compute.instances.insert` – VM created
   - `iam.serviceAccounts.create` – Service account created
   - `storage.buckets.create` – Bucket created

### Expected Result

- A list of administrative actions in your project
- Each entry shows who did what and when

### Learning Point

**Audit logs** provide a compliance trail. In regulated industries, you must prove who changed what. GCP keeps these logs by default.

---

## Step 8: Create a Logs-Based Metric

**Goal:** Turn log events into numbers you can alert on.

**Scenario:** You want to be notified whenever your VM produces an ERROR-level log. Logs-based metrics count those events.

### Steps

1. Go to **Logging → Logs-based Metrics**
2. Click **Create Metric**
3. Configure:
   - **Metric type:** Counter
   - **Name:** `vm-error-count`
   - **Description:** `Count of ERROR logs from VMs`
   - **Filter:**

```
resource.type="gce_instance"
severity>=ERROR
```

4. Click **Create Metric**

### Expected Result

- A new metric `vm-error-count` appears in the list
- It will increment whenever a VM emits an ERROR log

### Learning Point

**Logs → Metrics → Alerts.** You can't alert directly on "any error log," but you can create a metric that counts errors, then alert when that count is greater than zero.

---

## Step 9: Create an Alert from the Logs-Based Metric

**Goal:** Get notified when any VM error occurs.

### Steps

1. Go to **Monitoring → Alerting**
2. Click **Create Policy**
3. Add condition:
   - **Metric:** Search for `vm-error-count` (under "Log metrics")
   - **Condition:** `is above`
   - **Threshold:** `0`
4. Add notification channel (your email)
5. Name: `vm-errors-alert`
6. Click **Create Policy**

### Expected Result

- A new alert policy that fires when any VM logs an error
- You can test it by generating an error on the VM (e.g., a failed command) and checking if the alert fires

### Learning Point

Combining logs-based metrics with alerts gives you **event-driven monitoring**. You react to specific events, not just CPU spikes.

---

## Step 10: Apply Labels for Cost Governance

**Goal:** Tag resources so you can track costs by team, environment, or project.

---

### Steps

1. Go to **Compute Engine → VM Instances**
2. Click the **name** of your VM
3. Click **Edit** at the top
4. Scroll to **Labels**
5. Click **Add label** and add:

| Key          | Value       |
|--------------|-------------|
| `environment`| `dev`       |
| `owner`      | `cloud-team`|
| `cost-center`| `training`  |

6. Click **Save**

### Expected Result

- Labels appear on the VM details page
- You can filter VMs by label in the console

### Learning Point

Labels enable **cost allocation**. In enterprises, finance teams group billing by `cost-center` or `department` to see who is spending what.

---

## Step 11: Review Cost by Labels

**Goal:** See how labels appear in billing reports.

### Steps

1. Go to **Billing → Reports** (or **Billing → Cost table**)
2. Ensure you have billing enabled and some usage
3. In the report, look for **Group by** or **Labels**
4. Group by `cost-center` or `environment`
5. Observe how costs are broken down by label

### Expected Result

- Cost report shows spending grouped by your labels
- If usage is minimal (e.g., free tier), the numbers may be small

### Learning Point

This is how enterprises **manage cloud spend**. Labels turn raw billing data into actionable reports: "The dev team spent $X this month."

---

## Step 12: Organization Policy Overview (Conceptual)

**Goal:** Understand how GCP enforces governance rules at the organization level.

### Steps

1. Go to **IAM & Admin → Organization Policies**
2. Browse the list of available policies. Examples:
   - **Restrict public IP** – Prevent VMs from having public IPs
   - **Allowed regions** – Restrict where resources can be created
   - **Enforce encryption** – Require encryption for disks
3. Click any policy to see its description and possible values
4. **Do not change any policies** for this lab—just review

### Expected Result

- You see the types of constraints GCP can enforce
- Understanding these helps when designing secure, compliant environments

### Learning Point

**Organization policies** apply to all projects under an organization or folder. They prevent accidental misconfigurations (e.g., someone creating a VM in the wrong region).

---

## Step 13: Operational Hardening Checklist

**Goal:** Confirm your setup meets basic production-readiness criteria.

### Review and Check

| Item                    | Status |
|-------------------------|--------|
| Monitoring enabled      | ☐      |
| Dashboard created       | ☐      |
| CPU alert configured    | ☐      |
| Logs explored           | ☐      |
| Audit logs reviewed     | ☐      |
| Logs-based metric created | ☐    |
| Labels applied          | ☐      |
| Notification channel set | ☐     |

### Learning Point

This checklist defines **production readiness**. Before going live, ensure every item is done. Missing alerts or labels leads to blind spots and cost surprises.

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] Dashboard `infra-ops-dashboard` created
- [ ] CPU alert policy `high-cpu-alert` configured
- [ ] (Optional) Alert triggered and email received
- [ ] Logs Explorer used to view VM logs
- [ ] Audit logs reviewed
- [ ] Logs-based metric `vm-error-count` created
- [ ] Labels applied to VM for cost governance

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **Monitoring** | Detects problems early. Use dashboards for visibility and alerts for notification. |
| **Alerts** | Must be meaningful. Avoid alert fatigue by setting realistic thresholds. |
| **Logs** | Explain *why* something failed. Always check logs when troubleshooting. |
| **Audit logs** | Provide a compliance trail. Every admin action is recorded. |
| **Labels** | Essential for cost governance. Tag everything for billing and ownership. |
| **Governance** | Prevents chaos at scale. Policies and labels keep environments under control. |

---

## Next Steps

- Explore **Cloud Trace** for distributed tracing
- Set up **Uptime Checks** for HTTP/HTTPS monitoring
- Create **SLOs (Service Level Objectives)** for critical services
- Review **Security Command Center** for security posture

---

*Lab 6 – Monitoring, Logging & Governance | GCP Foundation Bootcamp*
