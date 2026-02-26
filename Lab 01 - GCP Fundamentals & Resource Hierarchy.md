# Lab 1: GCP Fundamentals & Resource Hierarchy

## Lab Overview

This **standalone lab** introduces you to Google Cloud Platform (GCP) and its core organizational model. You will learn how GCP structures resources and how to navigate the platform.

| Pillar | What It Does | Why It Matters |
|--------|--------------|----------------|
| **Resource Hierarchy** | Organizes resources: Organization → Folders → Projects | Enables governance, billing, and access control |
| **Regions & Zones** | Physical locations where resources run | Affects latency, availability, and compliance |
| **Console & CLI** | Two ways to manage GCP | Flexibility for different workflows and automation |

---

## Lab Objectives

By the end of this lab, you will:

- Navigate the Google Cloud Platform Console
- Understand regions, zones, and global services
- Create and manage a GCP project
- Explore the GCP resource hierarchy
- Use Cloud Shell and basic `gcloud` commands
- Validate project setup using CLI and Console

---

## Estimated Time

**60–90 minutes**

---

## Prerequisites (Standalone)

This lab is designed to run **on its own**. You need:

- A **Google account** (personal or corporate)
- **GCP free trial** enabled OR access to a billing account
- A modern browser (Chrome recommended)

---

## Understanding the Concepts

### What is the Resource Hierarchy?

Every GCP resource belongs to a **project**. Projects belong to **folders**, and folders belong to an **Organization** (for enterprises). This hierarchy enables:

- **Billing** – Each project can have its own billing account
- **IAM** – Permissions can be inherited from parent to child
- **Organization policies** – Rules that apply across projects

**Example:** A company might have folders for `Production` and `Development`, each containing multiple projects.

### What are Regions and Zones?

- **Region** – A geographic area (e.g., `us-central1`, `asia-south1`). Use regions for data residency and disaster recovery.
- **Zone** – A single data center within a region. VMs and disks are zonal; if a zone fails, resources in that zone are affected.

**Example:** `us-central1-a` is zone `a` in region `us-central1`. For high availability, spread VMs across multiple zones.

### What is Cloud Shell?

Cloud Shell is a **browser-based terminal** with `gcloud` pre-installed and authenticated. You can manage GCP without installing anything locally.

---

## Lab Scenario: Real-World Context

You are **setting up a new GCP environment** for your team. Your goals:

1. **Create a project** – Isolate resources and track costs
2. **Link billing** – Enable APIs and resource creation
3. **Use both Console and CLI** – Be comfortable with either interface
4. **Understand the hierarchy** – Know where resources live and how they inherit permissions

This lab walks you through the foundational setup.

---

## Step 1: Sign in to Google Cloud Console

**Goal:** Access the GCP Console and see the dashboard.

### Steps

1. Open a browser and go to [https://console.cloud.google.com](https://console.cloud.google.com)
2. Sign in with your Google account
3. Accept terms of service if prompted

### Expected Result

You see the **Google Cloud Console dashboard** with the project selector at the top.

### Learning Point

The Console is the primary UI for managing GCP. Bookmark it for quick access.

---

## Step 2: Explore GCP Global Infrastructure (Read-Only)

**Goal:** Understand where GCP resources can run.

### Steps

1. In the Console search bar, type **Regions**
2. Navigate to **Compute Engine → Settings → Regions**
3. Observe:
   - Available regions (e.g., `asia-south1`, `us-central1`)
   - Multiple zones per region (e.g., `asia-south1-a`, `asia-south1-b`)

### Expected Result

A list of regions and zones with their status.

### Learning Point

Regions and zones are **fixed**. You cannot create resources in arbitrary locations—only in GCP's published regions.

---

## Step 3: Understand the Resource Hierarchy (Visual)

**Goal:** See how projects, folders, and organizations are organized.

### Steps

1. Go to **IAM & Admin → Manage Resources**
2. Observe the hierarchy:
   - **Organization** (if you have one—enterprises typically do)
   - **Folders** (optional grouping)
   - **Projects**

If you don't see an Organization, that's normal for personal accounts. Organizations are used for enterprise governance.

### Expected Result

A tree view of your resource hierarchy. All resources belong to a project.

### Learning Point

The hierarchy enables **inheritance**. A parent's IAM policy can apply to all children. Design your hierarchy carefully.

---

## Step 4: Create a New GCP Project

**Goal:** Create a project for your lab work.

### Steps

1. Click the **project selector** (top-left, next to "Google Cloud")
2. Click **New Project**
3. Enter:
   - **Project Name:** `gcp-foundation-lab`
   - **Project ID:** Accept the auto-generated value (or customize if needed)
4. Select Organization or "No organization" as applicable
5. Click **Create**
6. Wait 30–60 seconds for provisioning

### Expected Result

The new project appears in the project selector. You can switch between projects by selecting it.

### Learning Point

Projects are the **primary unit of billing and resource isolation**. Use separate projects for dev, staging, and production.

---

## Step 5: Link Billing Account

**Goal:** Enable billing so you can create resources.

### Steps

1. Go to **Billing** (search for it or use the navigation menu)
2. Select or create a billing account
3. Link the billing account to your project `gcp-foundation-lab`
4. Confirm billing is enabled

### Expected Result

The project shows as linked to a billing account. Some services have free tiers; billing ensures you can use paid resources.

### Learning Point

**No resources can be created without billing.** Even free-tier resources require a billing account. Set budget alerts to avoid surprises.

---

## Step 6: Enable Cloud Shell

**Goal:** Access a terminal with `gcloud` pre-configured.

### Steps

1. Click **Activate Cloud Shell** (terminal icon in the top-right)
2. Wait for the terminal to initialize (first time may take 30–60 seconds)
3. You now have:
   - Pre-installed `gcloud` CLI
   - Temporary VM (no cost)
   - Authenticated session

### Expected Result

A terminal prompt appears at the bottom of the Console. You can run `gcloud` commands.

### Learning Point

Cloud Shell is **ephemeral**—files you create are lost when the session ends. Use it for quick tasks and exploration.

---

## Step 7: Set Active Project in gcloud

**Goal:** Configure `gcloud` to use your project by default.

### Steps

1. In Cloud Shell, run:

```bash
gcloud config list
```

2. If the project is not set, run:

```bash
gcloud config set project gcp-foundation-lab
```

Replace `gcp-foundation-lab` with your actual project ID if different.

3. Verify:

```bash
gcloud config get-value project
```

### Expected Result

The output shows your project ID. Subsequent commands will use this project by default.

### Learning Point

`gcloud config` sets **default values** for project, region, and zone. You can override them per command with flags.

---

## Step 8: Explore Projects Using CLI

**Goal:** List and describe projects via the command line.

### Steps

1. List accessible projects:

```bash
gcloud projects list
```

2. Describe your project:

```bash
gcloud projects describe gcp-foundation-lab
```

Replace with your project ID if different.

### Expected Result

A table of projects and a JSON description of the selected project.

### Learning Point

The same resources can be managed via **Console or CLI**. CLI is essential for automation and scripting.

---

## Step 9: Enable a Core API

**Goal:** Enable the Compute Engine API so you can create VMs later.

### Steps

1. Enable Compute Engine API:

```bash
gcloud services enable compute.googleapis.com
```

2. Verify:

```bash
gcloud services list --enabled
```

### Expected Result

`compute.googleapis.com` appears in the list of enabled APIs.

### Learning Point

GCP uses **opt-in APIs**. Each service (Compute, Storage, etc.) must be enabled per project before use.

---

## Step 10: Explore Regions & Zones via CLI

**Goal:** List regions and zones from the command line.

### Steps

1. List regions:

```bash
gcloud compute regions list
```

2. List zones (first 10):

```bash
gcloud compute zones list | head -10
```

3. Observe the region–zone mapping and naming (e.g., `us-central1-a`).

### Expected Result

Tables showing regions and zones with their status.

### Learning Point

Zones follow the pattern `REGION-ZONE`. For example, `asia-south1-a` is zone `a` in region `asia-south1`.

---

## Step 11: Navigate Core GCP Services (Console)

**Goal:** Familiarize yourself with key service areas.

### Steps

1. In the Console, open the **navigation menu** (☰)
2. Explore (read-only, do not create resources):
   - **IAM & Admin** – Users, roles, service accounts
   - **Compute Engine** – VMs, disks, images
   - **VPC Network** – Networks, firewalls
   - **Cloud Storage** – Buckets and objects
   - **Monitoring** – Metrics and logs

### Expected Result

You understand where to find each service. Some may show "Enable API" if not yet activated.

### Learning Point

GCP organizes services by **product**. Use the search bar to quickly find any service.

---

## Step 12: Validate Shared Responsibility Model (Conceptual)

**Goal:** Understand what Google manages vs. what you manage.

### Discussion Points

- **Google manages:** Physical infrastructure, hypervisor, networking hardware, physical security
- **You manage:** IAM, firewall rules, OS patching (on VMs), data encryption keys (if using CMEK)
- **IAM, networking, OS security** – All are your responsibility within the cloud boundary

### Learning Point

The **shared responsibility model** defines who secures what. GCP secures the platform; you secure your workloads and data.

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] Project `gcp-foundation-lab` created successfully
- [ ] Billing linked to project
- [ ] Cloud Shell accessed and working
- [ ] `gcloud` configured with correct project
- [ ] Compute Engine API enabled
- [ ] Regions and zones explored via CLI
- [ ] Console and CLI navigation understood

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **Resource hierarchy** | Organization → Folders → Projects. All resources belong to a project. |
| **Regions & zones** | Regions are geographic; zones are data centers. Choose based on latency and compliance. |
| **Console & CLI** | Both are equally powerful. Use Console for exploration, CLI for automation. |
| **Billing & APIs** | Billing must be linked; APIs must be enabled. Both are foundational. |
| **Shared responsibility** | Google secures the platform; you secure your workloads and access. |

---

## Next Steps

- **Lab 2** – Identity & Access Management (IAM)
- **Lab 3** – VPC Networking
- Explore **Cloud Shell** tutorials for more `gcloud` commands

---

*Lab 1 – GCP Fundamentals & Resource Hierarchy | GCP Foundation Bootcamp*
