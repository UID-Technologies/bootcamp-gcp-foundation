#  Lab 4: Compute Engine & Virtual Machines – Step-by-Step

##  Lab Objectives

By the end of this lab, participants will:

* Create and configure Compute Engine VMs in Google Cloud Platform
* Use **startup scripts** to automate VM configuration
* Understand VM sizing and disk options
* Create **instance templates**
* Deploy **Managed Instance Groups (MIGs)**
* Enable **autoscaling and self-healing**
* Validate VM health and scaling behavior

---

##  Estimated Time

**120 minutes**

---

##  Prerequisites

* Completed **Session 1, 2, and 3 labs**
* Custom VPC available (from Session 3)
* Owner / Compute Admin access
* Cloud Shell enabled
* Compute Engine API enabled

---

##  Lab Scenario (Real-World Context)

You are deploying a **stateless web application tier** that must:

* Auto-configure itself on startup
* Scale automatically based on load
* Recover automatically from VM failures
* Be production-ready and reproducible

---

![Image](https://docs.cloud.google.com/migrate/virtual-machines/docs/5.0/images/m2vm_architecture.svg)

![Image](https://docs.cloud.google.com/build/images/devops.png)

![Image](https://miro.medium.com/1%2AVu7tG2FlULvgWqlxa0GGDQ.png)

![Image](https://docs.cloud.google.com/static/compute/images/mig-instance-replacement-methods.svg)

![Image](https://i.sstatic.net/AOIcT.png)

---

##  Step 1: Review Compute Engine Quotas & Regions

1. Go to **Compute Engine → Settings**
2. Review:

   * Available regions/zones
   * CPU quotas
3. Choose region:

   * `asia-south1`

💡 **Learning Point:**
Always verify quotas before production deployment.

---

##  Step 2: Create a VM with Startup Script

1. Navigate to **Compute Engine → VM Instances**
2. Click **Create Instance**
3. Configure:

   * **Name:** `web-vm-1`
   * **Region:** `asia-south1`
   * **Zone:** `asia-south1-a`
   * **Machine type:** `e2-medium`
4. Under **Advanced options → Automation**
5. Add **Startup Script**:

```bash
#!/bin/bash
apt-get update
apt-get install -y apache2
echo "<h1>Welcome to GCP Compute Engine</h1>" > /var/www/html/index.html
systemctl start apache2
```

6. Network:

   * VPC: `custom-app-vpc`
   * Subnet: `app-subnet-asia`
7. Click **Create**

---

##  Step 3: Verify VM & Application

1. Wait for VM to start
2. Click **External IP**
3. Open in browser

 **Expected Result:**
Apache welcome page is displayed

---

##  Step 4: Understand VM Components (Checkpoint)

Discuss / review:

* Machine type (CPU & memory)
* Boot disk
* Network interface
* Service account
* Metadata & startup scripts

Trainer checkpoint 

---

##  Step 5: Create an Instance Template

1. Go to **Compute Engine → Instance Templates**
2. Click **Create Instance Template**
3. Configure:

   * **Name:** `web-template-v1`
   * **Machine type:** `e2-medium`
   * **Startup Script:** *(same as previous)*
   * **Network:** `custom-app-vpc`
4. Click **Create**

💡 **Learning Point:**
Instance templates are **immutable blueprints**

---

##  Step 6: Create a Managed Instance Group (MIG)

1. Navigate to **Compute Engine → Instance Groups**
2. Click **Create Instance Group**
3. Configure:

   * **Name:** `web-mig`
   * **Location:** Regional
   * **Region:** `asia-south1`
   * **Instance template:** `web-template-v1`
   * **Autoscaling:** Off (for now)
   * **Initial size:** 2
4. Click **Create**

---

##  Step 7: Verify MIG Instances

1. Open `web-mig`
2. Observe:

   * Two running VM instances
   * Auto-generated names
3. Click any instance and verify Apache page via IP

---

##  Step 8: Configure Health Checks

1. Go to **Compute Engine → Health Checks**
2. Click **Create Health Check**
3. Configure:

   * **Name:** `http-health-check`
   * **Protocol:** HTTP
   * **Port:** 80
   * **Request path:** `/`
4. Click **Create**

---

##  Step 9: Attach Health Check to MIG

1. Open **web-mig**
2. Click **Edit**
3. Enable **Auto-healing**
4. Select health check: `http-health-check`
5. Save changes

 Unhealthy VMs will now be **recreated automatically**

---

##  Step 10: Enable Autoscaling

1. Edit **web-mig**
2. Enable **Autoscaling**
3. Configure:

   * **Minimum instances:** 2
   * **Maximum instances:** 5
   * **Autoscaling metric:** CPU utilization
   * **Target CPU:** 60%
4. Save

---

##  Step 11: Simulate Load (Optional)

SSH into one instance:

```bash
sudo apt-get install -y stress
stress --cpu 2 --timeout 300
```

Observe autoscaling behavior in:

* **Instance Groups → web-mig**

---

##  Step 12: Validate Self-Healing

1. Manually stop one VM in the MIG
2. Observe:

   * MIG detects unhealthy instance
   * New VM is created automatically

💡 **Learning Point:**
Never manage individual VMs in a MIG.

---

##  Step 13: Verify via gcloud CLI

In Cloud Shell:

```bash
gcloud compute instances list
gcloud compute instance-groups managed list
gcloud compute instance-templates list
```

---

##  Step 14: Review Cost & Operational Best Practices

Discuss:

* Why autoscaling saves cost
* Why templates ensure consistency
* Why manual VM changes are avoided

---

##  Lab Validation Checklist

✔ VM created with startup script
✔ Web application auto-installed
✔ Instance template created
✔ Managed Instance Group deployed
✔ Health check enabled
✔ Autoscaling configured
✔ Self-healing validated

---

##  Key Takeaways

* Compute Engine provides full VM control
* Startup scripts automate configuration
* Instance templates ensure repeatability
* MIGs provide availability and resilience
* Autoscaling optimizes cost and performance

---
