#  Lab 2: Identity & Access Management (IAM) 

## Lab Objectives

By the end of this lab, participants will be able to:

* Understand IAM building blocks in Google Cloud Platform
* Create and manage IAM users and service accounts
* Assign predefined roles following least-privilege principles
* Validate access using Cloud Console and `gcloud`
* Audit IAM changes using Cloud Audit Logs

---

##  Estimated Time

**90–120 minutes**

---

## Prerequisites

* Completed **Session 1 – GCP Fundamentals lab**
* Active GCP project with billing enabled
* Owner or IAM Admin access on the project
* Cloud Shell enabled

---

## Lab Scenario (Real-World Context)

You are a **Cloud / Infrastructure Engineer** responsible for:

* Granting developers **read-only** access
* Granting operators **resource management** access
* Configuring a **service account** for VM workloads
* Ensuring **no excessive permissions** are granted

---

![Image](https://storage.googleapis.com/gweb-cloudblog-publish/images/getting-to-know-iam-flowchart-2od05.max-700x700.PNG)

---

![Image](https://miro.medium.com/1%2AdpVWy1ony8aYU1f1uqX8YA.jpeg)

---

![Image](https://docs.cloud.google.com/static/iam/img/policy-inheritance.svg)

---

![Image](https://docs.cloud.google.com/resource-manager/img/org-policy-inheritance.svg)

----


##  Step 1: Review Current IAM Configuration

1. Open **GCP Console**
2. Navigate to:
   **IAM & Admin → IAM**
3. Review existing principals:

   * Your user account
   * Any default service accounts
4. Note assigned roles (Owner, Editor, Viewer)

 **Learning Point:**
IAM policies are evaluated from **top to bottom with inheritance**

---

##  Step 2: Create a New IAM User (Developer)

> If using a personal account, simulate using another Google account.

1. Go to **IAM & Admin → IAM**
2. Click **Grant Access**
3. Enter:

   * **Principal:** [developer-user@gmail.com](mailto:developer-user@gmail.com)
   * **Role:** Viewer
4. Click **Save**

 **Expected Result:**
User appears with **Viewer** role

---

##  Step 3: Validate Least-Privilege Access (Conceptual)

Discuss / validate:

* Developer can **view resources**
* Cannot create, modify, or delete resources
* Suitable for:

  * Audits
  * Read-only dashboards
  * Support roles

⚠️ Avoid granting **Editor** unless absolutely required

---

##  Step 4: Create an Operations User with Scoped Access

1. Click **Grant Access**
2. Add:

   * **Principal:** [ops-user@gmail.com](mailto:ops-user@gmail.com)
   * **Role:** Compute Admin
3. Click **Save**

 **Best Practice:**
Use **service-specific predefined roles** instead of Editor

---

##  Step 5: Understand IAM Role Types (Checkpoint)

Confirm understanding:

* Primitive roles → broad (avoid in prod)
* Predefined roles → recommended
* Custom roles → compliance-driven

---

##  Step 6: Create a Service Account for Compute Workloads

1. Go to **IAM & Admin → Service Accounts**
2. Click **Create Service Account**
3. Enter:

   * **Name:** `vm-storage-sa`
   * **ID:** auto-generated
4. Click **Create and Continue**
5. Assign role:

   * **Storage Object Creator**
6. Click **Done**

 **Expected Result:**
Service account created with limited permissions

---

##  Step 7: Avoid Service Account Keys (Security Best Practice)

1. Open the service account
2. Go to **Keys** tab
3. Confirm:

   * No keys created

 **Security Rule:**
Use **attached service accounts**, not key files

---

##  Step 8: Assign Additional IAM Role at Resource Level

1. Navigate to **Cloud Storage**
2. Open an existing bucket (or create one if needed)
3. Go to **Permissions**
4. Click **Grant Access**
5. Add:

   * **Principal:** `vm-storage-sa`
   * **Role:** Storage Object Viewer
6. Save

 **Outcome:**
Service account has **bucket-scoped access**

---

## Step 9: Validate IAM Using gcloud CLI

In Cloud Shell, run:

```bash
gcloud projects get-iam-policy gcp-foundation-lab
```

Search for:

* Developer user
* Ops user
* Service account roles

 **Learning Point:**
IAM policies are stored as **bindings**

---

##  Step 10: Test Access Boundaries (Simulation)

Discuss expected behavior:

* Developer user cannot create VM
* Ops user can manage Compute Engine
* Service account can access Cloud Storage objects only

(Optional: actual login if accounts are available)

---

##  Step 11: Review IAM Policy Inheritance

1. Navigate to **IAM & Admin → Manage Resources**
2. Select the project
3. Observe inherited permissions (if Org/Folder exists)

Explain:

* Why inherited access cannot be overridden downward
* Importance of clean hierarchy design

---

##  Step 12: Audit IAM Changes Using Logs

1. Go to **Cloud Logging → Logs Explorer**
2. Filter by:

   * Resource type: `project`
   * Log name: `cloudaudit.googleapis.com/activity`
3. Look for:

   * IAM policy changes
   * Role assignments

 **Expected Result:**
Audit entries showing IAM modifications

---

##  Step 13: IAM Troubleshooting Checklist

If access fails, check:

* Correct role?
* Correct resource scope?
* API enabled?
* Correct principal?
* Audit logs for denial reason?

---

##  Lab Validation Checklist

✔ IAM users created
✔ Least-privilege roles assigned
✔ Service account created
✔ No service account keys used
✔ Resource-level IAM applied
✔ IAM verified via CLI
✔ IAM changes audited

---

##  Key Takeaways

* IAM is **identity + role + resource**
* Least privilege is mandatory
* Service accounts are for workloads
* Predefined roles are production-safe
* Audit logs are your security trail

---
