# Lab 1: GCP Fundamentals & Resource Hierarchy

##  Lab Objective

By the end of this lab, participants will:

* Navigate the Google Cloud Platform Console
* Understand regions, zones, and global services
* Create and manage a GCP project
* Explore the GCP resource hierarchy
* Use Cloud Shell and basic `gcloud` commands
* Validate project setup using CLI and Console

---

## Estimated Time

**60–90 minutes**

---

##  Prerequisites

* Google account (personal or corporate)
* GCP free trial enabled **OR** access to a training billing account
* Modern browser (Chrome recommended)

---

##  Architecture Context (Conceptual)

This lab aligns with the following concepts:

* GCP global infrastructure
* Organization → Folder → Project hierarchy
* Console + CLI interaction


----


![Image](https://www.researchgate.net/publication/337894279/figure/fig1/AS%3A882099414384641%401587320297905/Block-diagram-showing-the-different-services-of-Google-cloud-platform-when-used-as.png)

----

![Image](https://storage.googleapis.com/gweb-cloudblog-publish/images/infrastructure-2.max-2000x2000.png)

----

![Image](https://d33wubrfki0l68.cloudfront.net/eaddeba5e864fe63444fe247f7a7277b427e42c2/ed88b/gcpimages/02-architecture/resource-hierarchy-overview.png)

----

![Image](https://docs.cloud.google.com/resource-manager/img/cloud-hierarchy.svg)

---

## Step 1: Sign in to Google Cloud Console

1. Open browser and go to:
    [https://console.cloud.google.com](https://console.cloud.google.com)
2. Sign in with your Google account
3. Accept terms if prompted

**Expected Result:**
You see the **Google Cloud Console dashboard**

---

##  Step 2: Explore GCP Global Infrastructure (Read-Only)

1. In the Console search bar, type **“Regions”**
2. Navigate to:
   **Compute Engine → Settings → Regions**
3. Observe:

   * Available regions (asia-south1, us-central1, etc.)
   * Multiple zones per region

---

##  Step 3: Understand the Resource Hierarchy (Visual)

1. In Console, go to:
   **IAM & Admin → Manage Resources**
2. Observe the hierarchy:

   * Organization (if available)
   * Folders (may or may not exist)
   * Projects

If you don’t see an Organization:

* Explain enterprise vs personal account differences

 **Key Learning:**
All resources **must belong to a project**

---

##  Step 4: Create a New GCP Project

1. Click the **project selector** (top-left)
2. Click **New Project**
3. Enter:

   * **Project Name:** `gcp-foundation-lab`
   * **Project ID:** auto-generated (do not change)
4. Select Organization (if applicable)
5. Click **Create**

 Wait 30–60 seconds

 **Expected Result:**
Project is created and visible in project selector

---

##  Step 5: Link Billing Account

1. Go to **Billing**
2. Select or link a billing account
3. Confirm billing is enabled for the project

 **Important:**
No resources can be created without billing

---

##  Step 6: Enable Cloud Shell

1. Click **Activate Cloud Shell** (top-right)
2. Wait for terminal to initialize

You now have:

* Pre-installed `gcloud`
* Temporary VM
* Authenticated environment

 **Expected Result:**
Cloud Shell prompt appears

---

##  Step 7: Set Active Project in gcloud

Run the following command:

```bash
gcloud config list
```

If project is not set, run:

```bash
gcloud config set project gcp-foundation-lab
```

Verify:

```bash
gcloud config get-value project
```

---

##  Step 8: Explore Projects Using CLI

List accessible projects:

```bash
gcloud projects list
```

Describe your project:

```bash
gcloud projects describe gcp-foundation-lab
```

 **Learning Point:**
Same resources can be managed via **Console or CLI**

---

##  Step 9: Enable a Core API

Enable Compute Engine API:

```bash
gcloud services enable compute.googleapis.com
```

Verify:

```bash
gcloud services list --enabled
```

 **Expected Result:**
Compute Engine API appears in enabled list

---

##  Step 10: Explore Regions & Zones via CLI

Run:

```bash
gcloud compute regions list
```

Then:

```bash
gcloud compute zones list | head -10
```

Observe:

* Region-zone mapping
* Zonal naming convention

---

##  Step 11: Navigate Core GCP Services (Console)

In Console, explore:

* **IAM & Admin**
* **Compute Engine**
* **VPC Network**
* **Cloud Storage**
* **Monitoring**

Do NOT create resources yet
This is exploration only

---

##  Step 12: Validate Shared Responsibility Model

Trainer discussion + learner reflection:

* What Google manages?
* What customer manages?
* Where IAM, networking, OS security fit

---

##  Lab Validation Checklist

✔ Project created successfully
✔ Billing linked
✔ Cloud Shell accessed
✔ `gcloud` configured
✔ Compute API enabled
✔ Regions & zones explored
✔ Console + CLI navigation understood

---

##  Key Takeaways

* GCP is **project-centric**
* Resource hierarchy enables governance
* Console and CLI are equally powerful
* Regions & zones impact availability
* Billing & APIs are foundational

---