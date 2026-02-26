# Lab 4: Compute Engine & Virtual Machines

## Lab Overview

This **standalone lab** teaches you how to run and manage virtual machines (VMs) on Google Cloud Platform (GCP). You will build a scalable, self-healing web application tier.

| Pillar | What It Does | Why It Matters |
|--------|--------------|----------------|
| **Compute Engine** | VMs running on GCP infrastructure | Full control over OS and applications |
| **Instance Templates** | Immutable blueprints for VMs | Ensures consistency and repeatability |
| **Managed Instance Groups (MIG)** | Auto-scaling and self-healing VM pools | Availability and cost optimization |

---

## Lab Objectives

By the end of this lab, you will:

- Create and configure Compute Engine VMs in Google Cloud Platform
- Use **startup scripts** to automate VM configuration
- Understand VM sizing and disk options
- Create **instance templates**
- Deploy **Managed Instance Groups (MIGs)**
- Enable **autoscaling and self-healing**
- Validate VM health and scaling behavior

---

## Estimated Time

**120 minutes**

---

## Prerequisites (Standalone)

This lab is designed to run **on its own**. You need:

- A **Google Cloud account** (free trial or paid)
- **Owner** or **Compute Admin** role on a GCP project
- Cloud Shell enabled
- Compute Engine API enabled

**If you don't have a VPC yet:** Use the default VPC, or complete Lab 3 to create a custom VPC first. This lab works with either.

---

## Understanding the Concepts

### What is Compute Engine?

Compute Engine provides **virtual machines** on GCP. You choose the machine type (CPU, memory), disk, and network. VMs run your applications with full OS control.

**Example:** An `e2-medium` VM has 2 vCPUs and 4 GB RAM—suitable for a small web server.

### What are Startup Scripts?

Startup scripts run **once when a VM boots**. They automate installation and configuration (e.g., install Apache, copy files). No manual setup needed.

**Example:** A script that runs `apt-get install -y nginx` ensures every VM starts with Nginx installed.

### What are Managed Instance Groups (MIGs)?

A MIG is a **pool of identical VMs** managed as a group. You define a template; the MIG creates instances from it. MIGs support:

- **Autoscaling** – Add/remove VMs based on CPU or custom metrics
- **Self-healing** – Replace unhealthy VMs automatically
- **Rolling updates** – Deploy new versions without downtime

**Example:** A web app MIG with 2–5 instances scales up during peak traffic and replaces failed instances automatically.

---

## Lab Scenario: Real-World Context

You are deploying a **stateless web application tier** that must:

1. **Auto-configure on startup** – No manual installation
2. **Scale automatically** – Handle traffic spikes
3. **Recover from failures** – Replace unhealthy VMs
4. **Be reproducible** – Same configuration every time

This lab walks you through building that tier.

---

## Quick Setup (If You Don't Have a VPC)

**Skip this section if you already have a VPC (from Lab 3 or default).**

- Use the **default** VPC and subnet for simplicity, or
- Complete Lab 3 to create `custom-app-vpc` and `app-subnet-asia`

---

## Step 1: Review Compute Engine Quotas & Regions

**Goal:** Verify you can create VMs in your chosen region.

### Steps

1. Go to **Compute Engine → Settings**
2. Review:
   - Available regions and zones
   - CPU quotas (e.g., 8 vCPUs per region on free tier)
3. Choose a region (e.g., `asia-south1` or `us-central1`)

### Expected Result

You see quotas and regions. Ensure you have capacity for at least 2–4 vCPUs.

### Learning Point

**Always verify quotas** before production deployment. Request increases in advance if needed.

---

## Step 2: Create a VM with Startup Script

**Goal:** Launch a VM that auto-installs Apache and serves a web page.

### Steps

1. Navigate to **Compute Engine → VM Instances**
2. Click **Create Instance**
3. Configure:
   - **Name:** `web-vm-1`
   - **Region:** `asia-south1` (or your preferred region)
   - **Zone:** `asia-south1-a`
   - **Machine type:** `e2-medium` (2 vCPU, 4 GB RAM)
4. Under **Boot disk**, keep default (10 GB, Debian or Ubuntu)
5. Expand **Advanced options** → **Management** → **Automation**
6. In **Startup script**, paste:

```bash
#!/bin/bash
apt-get update
apt-get install -y apache2
echo "<h1>Welcome to GCP Compute Engine</h1>" > /var/www/html/index.html
systemctl start apache2
```

7. Under **Networking**:
   - **Network:** `custom-app-vpc` (or default)
   - **Subnet:** `app-subnet-asia` (or default)
8. Click **Create**

> **Note:** If using a custom VPC, add a firewall rule allowing TCP port 80 (HTTP) from `0.0.0.0/0` or your IP before testing the web page. Go to **VPC Network → Firewall → Create** and add `allow-http` with TCP:80.

### Expected Result

VM starts. Wait 2–3 minutes for the startup script to complete. Apache will be running.

### Learning Point

Startup scripts run as **root** on first boot. Use them for one-time setup. For ongoing config, use Configuration Management (e.g., Ansible) or container images.

---

## Step 3: Verify VM & Application

**Goal:** Confirm the web server is serving content.

### Steps

1. Wait for the VM to start (status: green checkmark)
2. Note the **External IP**
3. Open `http://EXTERNAL_IP` in a browser

### Expected Result

A page showing "Welcome to GCP Compute Engine" appears.

### Learning Point

If the page doesn't load, check: (1) Firewall allows TCP 80, (2) Startup script completed (check serial console or SSH and run `systemctl status apache2`).

---

## Step 4: Understand VM Components (Checkpoint)

**Goal:** Review key VM configuration elements.

### Summary

| Component | Purpose |
|-----------|---------|
| **Machine type** | CPU and memory (e2-small, e2-medium, etc.) |
| **Boot disk** | OS and persistent storage |
| **Network interface** | VPC, subnet, internal/external IP |
| **Service account** | Identity for GCP API calls |
| **Metadata & startup script** | Automation and configuration |

### Learning Point

Every component affects **cost and performance**. Right-size machine types; use appropriate disk types (SSD vs. HDD).

---

## Step 5: Create an Instance Template

**Goal:** Create a reusable blueprint for identical VMs.

### Steps

1. Go to **Compute Engine → Instance Templates**
2. Click **Create Instance Template**
3. Configure:
   - **Name:** `web-template-v1`
   - **Machine type:** `e2-medium`
   - **Boot disk:** Same as VM (e.g., 10 GB Debian)
   - **Networking:** Same VPC and subnet
4. Under **Management** → **Automation**, add the same startup script:

```bash
#!/bin/bash
apt-get update
apt-get install -y apache2
echo "<h1>Welcome to GCP Compute Engine</h1>" > /var/www/html/index.html
systemctl start apache2
```

5. Click **Create**

### Expected Result

A new template `web-template-v1` appears. It does not create a VM—it is a blueprint.

### Learning Point

Instance templates are **immutable**. To change config, create a new template version. MIGs use templates to create instances.

---

## Step 6: Create a Managed Instance Group (MIG)

**Goal:** Deploy a group of VMs from the template.

### Steps

1. Navigate to **Compute Engine → Instance Groups**
2. Click **Create Instance Group**
3. Configure:
   - **Name:** `web-mig`
   - **Location:** Regional
   - **Region:** `asia-south1`
   - **Instance template:** `web-template-v1`
   - **Autoscaling:** Off (for now)
   - **Number of instances:** 2
4. Click **Create**

### Expected Result

Two VMs are created from the template. They appear in the instance group.

### Learning Point

MIGs manage instances for you. **Do not** manually modify MIG instances—changes will be lost when the instance is recreated. Use templates for changes.

---

## Step 7: Verify MIG Instances

**Goal:** Confirm both instances are running and serving traffic.

### Steps

1. Open the instance group `web-mig`
2. Observe:
   - Two running VM instances
   - Auto-generated names (e.g., `web-mig-xxxx`)
3. Click any instance, note its external IP
4. Open `http://EXTERNAL_IP` in a browser

### Expected Result

Both instances show the same web page. They are identical copies.

### Learning Point

MIG instances are **stateless** and interchangeable. Use a load balancer (not covered here) to distribute traffic across them.

---

## Step 8: Configure Health Checks

**Goal:** Define what "healthy" means for the MIG.

### Steps

1. Go to **Compute Engine → Health Checks**
2. Click **Create Health Check**
3. Configure:
   - **Name:** `http-health-check`
   - **Protocol:** HTTP
   - **Port:** 80
   - **Request path:** `/`
4. Click **Create**

### Expected Result

A health check that probes HTTP on port 80. Unhealthy instances fail this check.

### Learning Point

Health checks are used by **load balancers** and **auto-healing**. The MIG will use this to determine if an instance should be replaced.

---

## Step 9: Attach Health Check to MIG (Auto-Healing)

**Goal:** Enable automatic replacement of unhealthy instances.

### Steps

1. Open the instance group `web-mig`
2. Click **Edit**
3. Under **Autohealing**, enable it
4. Select health check: `http-health-check`
5. Set **Initial delay** (e.g., 300 seconds) so new instances have time to boot
6. Click **Save**

### Expected Result

The MIG will now replace instances that fail the health check.

### Learning Point

**Auto-healing** keeps your application available. If Apache crashes or the VM hangs, the MIG recreates the instance automatically.

---

## Step 10: Enable Autoscaling

**Goal:** Scale the MIG based on CPU utilization.

### Steps

1. Edit the instance group `web-mig`
2. Enable **Autoscaling**
3. Configure:
   - **Minimum instances:** 2
   - **Maximum instances:** 5
   - **Autoscaling metric:** CPU utilization
   - **Target CPU utilization:** 60%
4. Click **Save**

### Expected Result

The MIG will add instances when CPU exceeds 60% and remove them when it drops. Minimum 2, maximum 5.

### Learning Point

**Autoscaling** optimizes cost and performance. You pay only for instances when needed. Set realistic min/max to avoid runaway scaling.

---

## Step 11: Simulate Load (Optional)

**Goal:** Trigger autoscaling by generating CPU load.

### Steps

1. SSH into one instance in the MIG
2. Run:

```bash
sudo apt-get update && sudo apt-get install -y stress
stress --cpu 2 --timeout 300
```

3. In **Instance Groups → web-mig**, watch the instance count
4. After a few minutes, a new instance may be added (if CPU stays above 60%)

### Expected Result

CPU spikes; autoscaler may add instances. When stress stops, instances may be removed (after cooldown).

### Learning Point

**Test autoscaling** before production. Understand cooldown periods and scaling behavior.

---

## Step 12: Validate Self-Healing (Optional)

**Goal:** See the MIG replace an unhealthy instance.

### Steps

1. In the MIG, **stop** one VM manually (Compute Engine → VM Instances → Stop)
2. Observe in the instance group:
   - MIG detects the instance is gone or unhealthy
   - A new instance is created automatically
3. Wait a few minutes for the new instance to become healthy

### Expected Result

The stopped instance is replaced. The MIG maintains the desired number of instances.

### Learning Point

**Never manage individual VMs in a MIG.** Stopping, resizing, or major changes should go through the MIG (e.g., rolling update). Manual changes are overwritten.

---

## Step 13: Verify via gcloud CLI

**Goal:** List resources via command line.

### Steps

1. In Cloud Shell, run:

```bash
gcloud compute instances list
gcloud compute instance-groups managed list
gcloud compute instance-templates list
```

### Expected Result

Tables showing your VMs, instance groups, and templates.

### Learning Point

CLI is useful for **automation** and **scripting**. Same resources as the Console.

---

## Step 14: Review Cost & Operational Best Practices (Conceptual)

**Goal:** Understand why this architecture saves cost and improves operations.

### Summary

| Practice | Benefit |
|----------|---------|
| **Autoscaling** | Pay only for instances when needed; scale down during low traffic |
| **Templates** | Consistency; no configuration drift |
| **Avoid manual VM changes** | MIG manages instances; manual changes are lost on recreate |
| **Health checks** | Unhealthy instances are replaced automatically |

### Learning Point

**Production readiness** means automation, consistency, and self-healing. Manual intervention should be the exception.

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] VM `web-vm-1` created with startup script
- [ ] Web application (Apache) auto-installed and accessible
- [ ] Instance template `web-template-v1` created
- [ ] Managed Instance Group `web-mig` deployed with 2 instances
- [ ] Health check `http-health-check` created
- [ ] Auto-healing enabled on MIG
- [ ] Autoscaling configured (2–5 instances, 60% CPU)
- [ ] (Optional) Self-healing validated

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **Compute Engine** | Full VM control. Choose machine type, disk, network. |
| **Startup scripts** | Automate configuration on first boot. |
| **Instance templates** | Immutable blueprints. Ensure repeatability. |
| **MIGs** | Availability and resilience. Autoscaling + self-healing. |
| **Autoscaling** | Optimizes cost and performance. Set min/max wisely. |

---

## Next Steps

- **Lab 5** – Cloud Storage (integrate with VMs)
- **Lab 6** – Monitoring, Logging & Governance (monitor the MIG)
- Explore **Load Balancing** to distribute traffic across MIG instances

---

*Lab 4 – Compute Engine & Virtual Machines | GCP Foundation Bootcamp*
