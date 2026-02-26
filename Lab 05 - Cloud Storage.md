# Lab 5: Cloud Storage

## Lab Overview

This **standalone lab** teaches you how to store and manage objects in Google Cloud Platform (GCP). You will learn bucket configuration, access control, and cost optimization.

| Pillar | What It Does | Why It Matters |
|--------|--------------|----------------|
| **Buckets** | Containers for objects (files) in a region | Organize and isolate data |
| **Storage Classes** | Standard, Nearline, Coldline, Archive | Balance cost vs. access frequency |
| **IAM & Lifecycle** | Who can access; automatic transitions | Security and cost control |

---

## Lab Objectives

By the end of this lab, you will:

- Understand Cloud Storage concepts in Google Cloud Platform
- Create and configure **Cloud Storage buckets**
- Apply **IAM-based access control**
- Enable **object versioning**
- Configure **lifecycle management rules**
- Validate **VM → Cloud Storage integration**
- Understand **encryption and governance best practices**

---

## Estimated Time

**90–120 minutes**

---

## Prerequisites (Standalone)

This lab is designed to run **on its own**. You need:

- A **Google Cloud account** (free trial or paid)
- **Owner** or **Storage Admin** role on a GCP project
- Cloud Shell enabled
- At least one Compute Engine VM (optional for full integration; can create in Quick Setup)

**If you don't have a VM or service account:** Follow the Quick Setup section. Lab 2 creates the service account; this lab can create one if needed.

---

## Understanding the Concepts

### What is Cloud Storage?

Cloud Storage is **object storage**—you store files (objects) in buckets. Unlike a file system, there are no folders; objects have names (keys) that can include slashes to simulate hierarchy.

**Example:** `gs://my-bucket/logs/2024/app.log` is an object with a name that looks like a path.

### What are Storage Classes?

Storage classes trade **cost for access frequency**:

| Class | Use Case | Cost |
|-------|----------|------|
| **Standard** | Frequently accessed data | Highest |
| **Nearline** | Accessed once a month | Lower |
| **Coldline** | Accessed once a quarter | Lower still |
| **Archive** | Long-term backup, rarely accessed | Lowest |

**Example:** Move old logs to Coldline after 30 days to save cost.

### What is Uniform Bucket-Level Access?

With **uniform bucket-level access**, IAM controls all access. Legacy ACLs (per-object permissions) are disabled. This simplifies security—one place to manage permissions.

**Example:** Grant `roles/storage.objectViewer` to a service account; it can read all objects in the bucket.

---

## Lab Scenario: Real-World Context

You are a **Cloud Infrastructure Engineer** tasked with:

1. **Storing application logs and artifacts** securely
2. **Ensuring only authorized workloads** can write data (service account, not public)
3. **Automatically managing data lifecycle** to reduce cost (move old data to cheaper classes)
4. **Meeting basic security and compliance** (no public access, versioning)

This lab walks you through setting that up.

---

## Quick Setup (If You Don't Have a VM or Service Account)

**Skip this section if you already have a VM and service account from Labs 2 and 4.**

1. **Create a service account** (Lab 2, Step 6): `vm-storage-sa` with Storage Object Creator
2. **Create a VM** (Lab 4, Step 2) and attach `vm-storage-sa` as its service account
3. Or: Use Cloud Shell for uploads (uses your user identity)

---

## Step 1: Review Cloud Storage Concepts (Checkpoint)

**Goal:** Confirm understanding before hands-on work.

### Summary

| Concept | Description |
|---------|-------------|
| **Bucket** | Top-level container; globally unique name; belongs to a project |
| **Object** | File + metadata; identified by name (key) |
| **Storage class** | Standard, Nearline, Coldline, Archive |
| **IAM vs ACLs** | IAM = project/bucket level; ACLs = per-object (legacy) |
| **Versioning** | Keep old versions when object is overwritten |
| **Lifecycle rules** | Auto-transition or delete based on age |

### Learning Point

Cloud Storage is **object-based**, not file-based. No traditional directories—use naming conventions (e.g., `folder/subfolder/file.txt`).

---

## Step 2: Create a Cloud Storage Bucket

**Goal:** Create a private, regional bucket.

### Steps

1. Go to **Cloud Storage → Buckets**
2. Click **Create**
3. Configure:
   - **Name:** `gcp-foundation-storage-UNIQUE_ID` (bucket names are globally unique; add random suffix like your project ID or initials)
   - **Location type:** Region
   - **Region:** `asia-south1` (or your preferred region)
4. Click **Continue**
5. Choose **Standard** storage class
6. Under **Access control**, select **Uniform**
7. Under **Protection**, enable **Public access prevention**
8. Click **Create**

### Expected Result

A private, regionally located bucket. No public access.

### Learning Point

**Bucket names are globally unique** across all of GCP. Use a suffix (project ID, random string) to avoid conflicts.

---

## Step 3: Configure Storage Class & Access Control

**Goal:** Confirm bucket is configured for security.

### Steps

1. Open the bucket
2. Go to the **Configuration** tab
3. Verify:
   - **Default storage class:** Standard
   - **Access control:** Uniform bucket-level access
   - **Public access prevention:** Enforced

### Expected Result

All settings match the secure configuration. No public access.

### Learning Point

**Uniform bucket-level access** simplifies security. One IAM policy for the whole bucket. Disable ACLs for new buckets.

---

## Step 4: Review Bucket Configuration

**Goal:** Understand bucket settings and where to find them.

### Summary

| Setting | Location | Purpose |
|---------|----------|---------|
| Location | Configuration | Data residency |
| Storage class | Configuration | Default for new objects |
| Public access | Configuration | Prevent accidental exposure |
| IAM | Permissions tab | Who can access |

### Learning Point

Review configuration **before** uploading sensitive data. Location cannot be changed after creation.

---

## Step 5: Enable Object Versioning

**Goal:** Protect against accidental overwrites and deletions.

### Steps

1. Open the bucket
2. Go to the **Configuration** tab
3. Under **Object versioning**, click **Edit**
4. Enable **Object versioning**
5. Click **Save**

### Expected Result

Versioning is on. Overwriting an object creates a new version; the old one is retained.

### Learning Point

**Versioning** protects against accidental overwrites and some ransomware. Note: storage cost increases (you pay for all versions). Use lifecycle rules to delete old versions.

---

## Step 6: Configure Lifecycle Management Rules

**Goal:** Automatically move old data to cheaper storage and delete very old data.

### Steps

1. Open the bucket
2. Go to the **Lifecycle** tab
3. Click **Add a rule**
4. **Rule 1 – Move to Coldline:**
   - **Action:** Set storage class to Coldline
   - **Condition:** Age > 30 days
   - Click **Continue** → **Create**
5. **Rule 2 – Delete old versions (optional):**
   - Click **Add a rule**
   - **Action:** Delete object
   - **Condition:** Age > 365 days (and optionally "with noncurrent version")
   - Click **Continue** → **Create**

### Expected Result

Two lifecycle rules. Objects older than 30 days move to Coldline; objects older than 365 days are deleted (or old versions deleted).

### Learning Point

**Lifecycle rules** automate cost optimization. You don't need to manually move or delete old data. Plan retention based on compliance requirements.

---

## Step 7: Assign IAM Access to Service Account

**Goal:** Allow the VM's service account to upload objects.

### Steps

1. Open the bucket
2. Go to the **Permissions** tab
3. Click **Grant Access**
4. Add:
   - **Principal:** `vm-storage-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com`
   - **Role:** `Storage Object Creator`
5. Click **Save**

Replace `YOUR_PROJECT_ID` with your project ID. If you don't have this service account, create it in Lab 2 or use your user account for Cloud Shell uploads.

### Expected Result

The service account can upload objects but not delete them (Storage Object Creator = write only).

### Learning Point

**Least privilege:** Creator allows upload but not delete. For read-only, use Storage Object Viewer. For both, use Storage Object Admin (only if needed).

---

## Step 8: Verify No Public Access

**Goal:** Confirm the bucket is not exposed to the internet.

### Steps

1. In bucket **Configuration**, verify:
   - **Public access prevention:** Enforced
2. In **Permissions**, confirm:
   - No `allUsers` or `allAuthenticatedUsers` bindings

### Expected Result

Only specific principals (you, service accounts) have access. No public access.

### Learning Point

**Never expose buckets publicly** unless explicitly required (e.g., static website hosting). Use signed URLs or IAM for controlled access.

---

## Step 9: Upload Objects via Cloud Shell

**Goal:** Upload a file using the CLI.

### Steps

1. In **Cloud Shell**, run:

```bash
echo "Hello from GCP Cloud Storage Lab" > sample.txt
gsutil cp sample.txt gs://YOUR_BUCKET_NAME/
```

Replace `YOUR_BUCKET_NAME` with your bucket name.

2. Verify:

```bash
gsutil ls gs://YOUR_BUCKET_NAME/
```

### Expected Result

The file appears in the bucket. You can also see it in the Console under Objects.

### Learning Point

`gsutil` is the CLI for Cloud Storage. It uses your current `gcloud` credentials. For service accounts, use a VM with the attached identity.

---

## Step 10: Test Object Versioning

**Goal:** See how versioning retains old versions.

### Steps

1. Overwrite the same object:

```bash
echo "Updated content" > sample.txt
gsutil cp sample.txt gs://YOUR_BUCKET_NAME/sample.txt
```

2. List all versions:

```bash
gsutil ls -a gs://YOUR_BUCKET_NAME/sample.txt
```

### Expected Result

Two versions of `sample.txt` are listed. The older version is retained (noncurrent).

### Learning Point

**Versioning** keeps history. You can restore a previous version by copying it over the current one. Use lifecycle rules to delete old versions and control cost.

---

## Step 11: Access Cloud Storage from Compute Engine VM

**Goal:** Upload from a VM using the attached service account (no keys).

### Steps

1. SSH into your VM (from Lab 4 or any VM with `vm-storage-sa` attached)
2. Run:

```bash
echo "Log from VM" > vm-log.txt
gsutil cp vm-log.txt gs://YOUR_BUCKET_NAME/
```

### Expected Result

Upload succeeds. The VM uses the attached service account—no key file needed.

### Learning Point

**Attached service accounts** are the preferred way for workloads to access GCP. No keys to manage or leak. Ensure the VM has the correct service account and the bucket grants it access.

---

## Step 12: Validate Access Boundaries

**Goal:** Confirm least privilege—service account cannot delete.

### Steps

1. From the VM, try to delete:

```bash
gsutil rm gs://YOUR_BUCKET_NAME/vm-log.txt
```

### Expected Result

**Permission denied.** Storage Object Creator does not include delete permission.

### Learning Point

**Least privilege** means granting only what's needed. Creator = upload; Viewer = read; Admin = full control. Use Creator for upload-only workloads.

---

## Step 13: Review Cloud Storage Logs

**Goal:** See how object operations are auditable.

### Steps

1. Go to **Logging → Logs Explorer**
2. Filter by:
   - **Resource type:** `gcs_bucket`
   - Or use: `resource.type="gcs_bucket"`
3. Look for entries such as:
   - `storage.objects.create`
   - `storage.objects.get`
   - Identity (service account or user) in the log

### Expected Result

Log entries showing object creation and access. Each operation is recorded.

### Learning Point

**Every object operation is auditable.** Use logs for compliance, security investigations, and troubleshooting.

---

## Step 14: Encryption Overview (Conceptual)

**Goal:** Understand encryption options.

### Summary

| Type | Description |
|------|-------------|
| **Google-managed (default)** | GCP encrypts data at rest automatically. No action needed. |
| **Customer-managed (CMEK)** | You provide keys via Cloud KMS. For compliance requirements. |
| **Customer-supplied (CSEK)** | You provide keys per request. Rare; complex to manage. |

### Learning Point

**Default encryption** is sufficient for most use cases. Use CMEK when compliance mandates key control. No CMEK implementation required for this foundation lab.

---

## Step 15: Review Governance & Cost Controls

**Goal:** Confirm your setup meets best practices.

### Checklist

- [ ] Uniform bucket-level access enabled
- [ ] Lifecycle rules configured (Coldline, delete)
- [ ] IAM scoped to service account (not broad roles)
- [ ] No public exposure
- [ ] Region aligned with compute (reduce egress cost)
- [ ] Versioning enabled (with lifecycle to manage old versions)

### Learning Point

**Governance** prevents cost overruns and security issues. Review bucket settings periodically.

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] Bucket created with unique name
- [ ] Uniform access enabled
- [ ] Public access prevention on
- [ ] Versioning enabled
- [ ] Lifecycle rules configured
- [ ] IAM access scoped to service account
- [ ] VM → bucket integration validated (upload works)
- [ ] Unauthorized actions blocked (delete denied)

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **Object storage** | Buckets hold objects. No traditional file system. |
| **IAM** | Primary security mechanism. Use uniform bucket-level access. |
| **Service accounts** | For workloads. Attached identity, no keys. |
| **Lifecycle rules** | Control long-term cost. Move to Coldline/Archive; delete when no longer needed. |
| **Observability** | Every operation is logged. Use for audit and compliance. |

---

## Next Steps

- **Lab 6** – Monitoring, Logging & Governance (monitor bucket access)
- Explore **Signed URLs** for temporary public access
- Explore **Cloud CDN** for serving static content with low latency

---

*Lab 5 – Cloud Storage | GCP Foundation Bootcamp*
