# Lab 2: Identity & Access Management (IAM)

## Lab Overview

This **standalone lab** teaches you how to control who can do what in Google Cloud Platform (GCP). You will learn the core building blocks of access control.

| Pillar | What It Does | Why It Matters |
|--------|--------------|----------------|
| **Principals** | Users, groups, and service accounts that need access | Identifies *who* is requesting access |
| **Roles** | Bundles of permissions (e.g., Viewer, Compute Admin) | Defines *what* they can do |
| **Resource scope** | Project, folder, or organization level | Controls *where* access applies |

---

## Lab Objectives

By the end of this lab, you will:

- Understand IAM building blocks in Google Cloud Platform
- Create and manage IAM users and service accounts
- Assign predefined roles following least-privilege principles
- Validate access using Cloud Console and `gcloud`
- Audit IAM changes using Cloud Audit Logs

---

## Estimated Time

**90–120 minutes**

---

## Prerequisites (Standalone)

This lab is designed to run **on its own**. You need:

- A **Google Cloud account** (free trial or paid)
- **Owner** or **IAM Admin** role on a GCP project
- Cloud Shell enabled

**If you don't have a project yet:** Complete Lab 1 (GCP Fundamentals) or create a project and link billing in the Console.

---

## Understanding the Concepts

### What is IAM?

IAM (Identity and Access Management) answers: *"Who can do what on which resource?"*

- **Who** – Principals (user, group, service account)
- **What** – Permissions (actions like `compute.instances.create`)
- **Which** – Resource scope (project, folder, bucket, etc.)

**Example:** Granting `roles/viewer` to `alice@company.com` on a project lets Alice view all resources in that project but not create or delete them.

### What are Roles?

Roles are **bundles of permissions**. GCP provides:

- **Primitive roles** – Owner, Editor, Viewer. Broad and rarely recommended for production.
- **Predefined roles** – Granular (e.g., `roles/compute.admin`, `roles/storage.objectViewer`). Use these.
- **Custom roles** – You define permissions. Use when compliance requires it.

**Example:** `roles/storage.objectCreator` allows uploading objects but not deleting them—good for a backup service.

### What are Service Accounts?

Service accounts are identities for **workloads** (VMs, Cloud Functions, etc.), not humans. They allow applications to call GCP APIs without user credentials.

**Example:** A VM with an attached service account can write to Cloud Storage without storing keys—more secure than key files.

---

## Lab Scenario: Real-World Context

You are a **Cloud Engineer** responsible for:

1. **Granting developers read-only access** – They can view resources but not change them
2. **Granting operators scoped access** – They can manage Compute Engine but not Storage
3. **Configuring a service account** for VM workloads to access Cloud Storage
4. **Ensuring no excessive permissions** – Follow least privilege

This lab walks you through setting up exactly that.

---

## Quick Setup (If You Don't Have a Project)

**Skip this section if you already have a GCP project with billing.**

1. Create a project (see Lab 1, Step 4)
2. Link billing
3. Enable Compute Engine and Cloud Storage APIs
4. Proceed to Step 1

---

## Step 1: Review Current IAM Configuration

**Goal:** See who has access to your project and what roles they have.

### Steps

1. Open **GCP Console**
2. Navigate to **IAM & Admin → IAM**
3. Review existing principals:
   - Your user account
   - Default Compute Engine service account (if Compute API is enabled)
4. Note the assigned roles (e.g., Owner, Editor)

### Expected Result

A table of principals and their roles. You see how IAM policies are currently configured.

### Learning Point

IAM policies are evaluated **top to bottom with inheritance**. A role at the organization level applies to all projects underneath.

---

## Step 2: Create a New IAM User (Developer)

**Goal:** Grant a user read-only (Viewer) access.

> If using a personal account, use another Google account (e.g., a colleague's) or create a test account. For solo practice, you can skip adding a second user and just understand the steps.

### Steps

1. Go to **IAM & Admin → IAM**
2. Click **Grant Access**
3. Enter:
   - **Principal:** `developer-user@gmail.com` (replace with actual email)
   - **Role:** `Viewer`
4. Click **Save**

### Expected Result

The user appears in the IAM list with the **Viewer** role.

### Learning Point

**Viewer** allows viewing all resources but no create, modify, or delete. Suitable for audits, dashboards, and support roles.

---

## Step 3: Validate Least-Privilege Access (Conceptual)

**Goal:** Understand what Viewer can and cannot do.

### Discussion

- **Viewer can:** List VMs, view bucket contents, see logs, read configurations
- **Viewer cannot:** Create VMs, delete buckets, change IAM, modify firewall rules
- **Avoid** granting Editor unless the user truly needs to create and delete resources

### Learning Point

**Least privilege** means granting the minimum access needed. Start with Viewer; add more only when required.

---

## Step 4: Create an Operations User with Scoped Access

**Goal:** Grant a user Compute Admin (not full Editor) for VM management.

### Steps

1. In **IAM & Admin → IAM**, click **Grant Access**
2. Add:
   - **Principal:** `ops-user@gmail.com` (replace with actual email)
   - **Role:** `Compute Admin`
3. Click **Save**

### Expected Result

The user has **Compute Admin**—can manage VMs, disks, and instance groups but not Storage, IAM, or Networking.

### Learning Point

Use **service-specific predefined roles** instead of Editor. Compute Admin is sufficient for VM operations without granting Storage or IAM access.

---

## Step 5: Understand IAM Role Types (Checkpoint)

**Goal:** Confirm understanding of role types.

### Summary

| Type | Use Case |
|------|----------|
| **Primitive** (Owner, Editor, Viewer) | Quick setup; avoid in production |
| **Predefined** (e.g., Compute Admin) | Recommended for most scenarios |
| **Custom** | When compliance requires specific permission sets |

### Learning Point

Predefined roles are **production-safe** and maintained by Google. Custom roles require ongoing maintenance.

---

## Step 6: Create a Service Account for Compute Workloads

**Goal:** Create a service account that VMs can use to access Cloud Storage.

### Steps

1. Go to **IAM & Admin → Service Accounts**
2. Click **Create Service Account**
3. Enter:
   - **Name:** `vm-storage-sa`
   - **ID:** Auto-generated (e.g., `vm-storage-sa`)
4. Click **Create and Continue**
5. Assign role: **Storage Object Creator**
6. Click **Done**

### Expected Result

A new service account `vm-storage-sa@PROJECT_ID.iam.gserviceaccount.com` with Storage Object Creator at the project level.

### Learning Point

Service accounts are for **workloads**, not humans. Attach them to VMs, Cloud Functions, or GKE pods.

---

## Step 7: Avoid Service Account Keys (Security Best Practice)

**Goal:** Confirm no keys are created—use attached identities instead.

### Steps

1. Open the service account `vm-storage-sa`
2. Go to the **Keys** tab
3. Confirm: **No keys created**

### Expected Result

The Keys tab shows no keys. The service account will use **attached identity** when assigned to a VM.

### Learning Point

**Never create service account keys** unless required (e.g., local development). Attached service accounts are more secure—no keys to leak.

---

## Step 8: Assign Additional IAM Role at Resource Level

**Goal:** Grant the service account bucket-scoped access (more restrictive than project-level).

### Steps

1. Navigate to **Cloud Storage → Buckets**
2. Create a bucket if needed (or use an existing one)
3. Click the bucket name
4. Go to the **Permissions** tab
5. Click **Grant Access**
6. Add:
   - **Principal:** `vm-storage-sa@PROJECT_ID.iam.gserviceaccount.com`
   - **Role:** `Storage Object Viewer`
7. Click **Save**

### Expected Result

The service account can read objects in this bucket. Combined with project-level Storage Object Creator, it can also upload—scoped to this bucket.

### Learning Point

**Resource-level IAM** overrides or narrows project-level access. Use it to limit service accounts to specific buckets.

---

## Step 9: Validate IAM Using gcloud CLI

**Goal:** View IAM policy via command line.

### Steps

1. In Cloud Shell, run:

```bash
gcloud projects get-iam-policy YOUR_PROJECT_ID --format=json
```

Replace `YOUR_PROJECT_ID` with your project ID.

2. Search the output for:
   - Developer user (Viewer)
   - Ops user (Compute Admin)
   - Service account bindings

### Expected Result

JSON output showing all IAM bindings. Each binding has `role` and `members`.

### Learning Point

IAM policies are stored as **bindings**—each binding maps a role to one or more principals.

---

## Step 10: Test Access Boundaries (Simulation)

**Goal:** Understand expected behavior of each principal.

### Expected Behavior

| Principal | Can Do | Cannot Do |
|-----------|--------|-----------|
| Developer (Viewer) | View all resources | Create VM, delete bucket |
| Ops (Compute Admin) | Manage VMs, disks | Manage Storage, IAM |
| Service account | Read/upload to assigned bucket | Delete objects (if only Creator+Viewer), access other buckets |

(Optional: Log in as each user to validate, if accounts are available.)

### Learning Point

**Test access** before production. Simulate or actually log in to verify least privilege works as expected.

---

## Step 11: Review IAM Policy Inheritance

**Goal:** Understand how permissions flow from parent to child.

### Steps

1. Navigate to **IAM & Admin → Manage Resources**
2. Select your project
3. If you have an Organization or Folder, observe inherited permissions

### Discussion

- Permissions at the **organization** level apply to all projects and folders below
- Permissions at the **project** level apply only to that project
- You **cannot override** inherited access at a lower level—only add more

### Learning Point

**Hierarchy design matters.** Place broad roles (e.g., Viewer for auditors) at the folder level; place specific roles at the project level.

---

## Step 12: Audit IAM Changes Using Logs

**Goal:** See how every IAM change is recorded for compliance.

### Steps

1. Go to **Logging → Logs Explorer**
2. Use the **Log name** dropdown and select **Cloud Audit Logs → Activity**
3. Or enter filter:

```
protoPayload.methodName:"SetIamPolicy" OR protoPayload.methodName:"SetProjectIamPolicy"
```

4. Look for entries showing IAM policy changes and who made them

### Expected Result

Audit entries showing IAM modifications with timestamp, principal, and action.

### Learning Point

**Audit logs** are your security trail. Every IAM change is recorded. Use them for compliance and incident investigation.

---

## Step 13: IAM Troubleshooting Checklist

**Goal:** Know what to check when access fails.

### Checklist

When access is denied, verify:

- [ ] Correct role assigned?
- [ ] Correct resource scope (project vs. bucket)?
- [ ] Required API enabled?
- [ ] Correct principal (user vs. service account)?
- [ ] Audit logs for denial reason?

### Learning Point

Most IAM issues are **misconfiguration**—wrong role, wrong scope, or API not enabled. Check these first.

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] IAM users created (Developer, Ops)
- [ ] Least-privilege roles assigned (Viewer, Compute Admin)
- [ ] Service account `vm-storage-sa` created
- [ ] No service account keys used
- [ ] Resource-level IAM applied to bucket
- [ ] IAM verified via `gcloud`
- [ ] IAM changes visible in audit logs

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **IAM model** | Identity + Role + Resource. Who, what, where. |
| **Least privilege** | Grant minimum access. Start with Viewer. |
| **Service accounts** | For workloads, not humans. Use attached identity, not keys. |
| **Predefined roles** | Production-safe. Prefer over primitive roles. |
| **Audit logs** | Every IAM change is recorded. Essential for compliance. |

---

## Next Steps

- **Lab 3** – VPC Networking
- **Lab 5** – Cloud Storage (uses the service account created here)
- Explore **Custom roles** for compliance-driven permission sets

---

*Lab 2 – Identity & Access Management (IAM) | GCP Foundation Bootcamp*
