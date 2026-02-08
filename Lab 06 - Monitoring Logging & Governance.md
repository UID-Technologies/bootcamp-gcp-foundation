#  Lab 6: Monitoring, Logging & Governance – Step-by-Step

##  Lab Objectives

By the end of this lab, participants will:

* Use **Cloud Monitoring** to observe VM and storage metrics
* Create **dashboards and alert policies**
* Explore **Cloud Logging** for troubleshooting and auditing
* Create **logs-based metrics**
* Apply **labels for cost governance**
* Understand **organization policies and operational hardening**

All tasks are performed in Google Cloud Platform.

---

##  Estimated Time

**90–120 minutes**

---

##  Prerequisites

* Completed **Sessions 1–5 labs**
* At least one running Compute Engine VM
* Cloud Storage bucket created
* Owner / Monitoring Admin access
* Cloud Shell enabled

---

##  Lab Scenario (Real-World Context)

You are responsible for **operating a production workload**:

* Detect performance issues early
* Receive alerts before outages
* Investigate issues using logs
* Enforce governance for cost and security
* Leave an audit trail for compliance

---

![Image](https://docs.cloud.google.com/static/architecture/images/hybrid-and-multi-cloud-monitoring-and-logging-patterns-7.svg)

![Image](https://docs.cloud.google.com/static/logging/docs/images/logs-explorer-interface.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AgP1M5_Er9ZyLX2XX.jpg)

![Image](https://miro.medium.com/1%2A7CGus45P_yJFZxoR6KxuNQ.png)

![Image](https://images-2-gvwk7ffjaa-uc.a.run.app/?imageID=ckj7en223akg009n566g.png)

---

##  Step 1: Verify Monitoring Is Enabled

1. Go to **Monitoring**
2. Open **Metrics Explorer**
3. Confirm Compute Engine metrics are visible

💡 **Learning Point:**
Monitoring is **enabled by default** but must be configured meaningfully.

---

##  Step 2: Explore VM Metrics

1. In **Monitoring → Metrics Explorer**
2. Select:

   * Resource type: `VM Instance`
   * Metric: `CPU utilization`
3. Choose your VM
4. Observe CPU usage graph

**Expected Result:**
Live CPU utilization data is visible

---

##  Step 3: Create a Custom Dashboard

1. Go to **Monitoring → Dashboards**
2. Click **Create Dashboard**
3. Add widget:

   * Line chart
   * Metric: CPU utilization
   * Resource: VM Instance
4. Add another widget:

   * Network bytes received
5. Save dashboard as:
   **`infra-ops-dashboard`**

💡 Dashboards provide **at-a-glance system health**.

---

## Step 4: Create an Alert Policy (CPU)

1. Go to **Monitoring → Alerting**
2. Click **Create Policy**
3. Configure condition:

   * Metric: CPU utilization
   * Threshold: > 60%
   * Duration: 1 minute
4. Configure notification:

   * Email (your email)
5. Name policy:
   **`high-cpu-alert`**
6. Create policy

 Alert is now active.

---

## Step 5: Trigger Alert (Optional)

1. SSH into VM
2. Run:

```bash
sudo apt-get install -y stress
stress --cpu 2 --timeout 180
```

3. Observe:

   * CPU spike
   * Alert triggered
   * Email notification received

💡 **Learning Point:**
Alerts should be **actionable**, not noisy.

---

##  Step 6: Explore Cloud Logging

1. Go to **Logging → Logs Explorer**
2. Filter:

   * Resource type: `gce_instance`
3. Observe:

   * System logs
   * Startup logs
   * SSH access logs

Logs are the **first stop for troubleshooting**.

---

##  Step 7: Review Audit Logs

1. In Logs Explorer, filter:

   * Log name: `cloudaudit.googleapis.com/activity`
2. Look for:

   * IAM changes
   * Resource creation
   * Policy updates

 Every administrative action is recorded.

---

##  Step 8: Create a Logs-Based Metric

1. Go to **Logging → Logs-based Metrics**
2. Click **Create Metric**
3. Configure:

   * Name: `vm-error-count`
   * Filter:

     ```
     resource.type="gce_instance"
     severity>=ERROR
     ```
4. Create metric

💡 This converts **logs → metrics → alerts**.

---

##  Step 9: Create Alert from Logs-Based Metric

1. Go to **Monitoring → Alerting**
2. Create policy using:

   * Metric: `vm-error-count`
   * Threshold: > 0
3. Save policy

 Now errors can trigger alerts automatically.

---

##  Step 10: Apply Labels for Governance

1. Go to **Compute Engine → VM Instances**
2. Edit your VM
3. Add labels:

   * `environment=dev`
   * `owner=cloud-team`
   * `cost-center=training`
4. Save

💡 Labels enable **cost tracking and ownership clarity**.

---

##  Step 11: Review Cost by Labels

1. Go to **Billing → Reports**
2. Group by **Labels**
3. Observe cost distribution

 This is how enterprises manage cloud spend.

---

##  Step 12: Organization Policy Overview (Conceptual)

Navigate to:
**IAM & Admin → Organization Policies**

Review example policies:

* Restrict public IP usage
* Restrict allowed regions
* Enforce encryption

⚠️ No policy changes required for this lab.

---

##  Step 13: Operational Hardening Checklist

Review and confirm:

* Monitoring enabled
* Alerts configured
* Logs centralized
* Audit logs reviewed
* Labels applied
* Least privilege enforced

This checklist defines **production readiness**.

---

##  Lab Validation Checklist

✔ Dashboard created
✔ CPU alert configured
✔ Alert triggered successfully
✔ Logs explored
✔ Audit logs reviewed
✔ Logs-based metric created
✔ Labels applied for cost governance

---

##  Key Takeaways

* Monitoring detects problems early
* Alerts must be meaningful
* Logs explain *why* something failed
* Audit logs provide compliance trail
* Labels are essential for cost governance
* Governance prevents chaos at scale

---
