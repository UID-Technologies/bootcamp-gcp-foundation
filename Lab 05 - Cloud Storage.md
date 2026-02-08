#  Lab 5: Cloud Storage – Step-by-Step

##  Lab Objectives

By the end of this lab, participants will:

* Understand Cloud Storage concepts in Google Cloud Platform
* Create and configure **Cloud Storage buckets**
* Apply **IAM-based access control**
* Enable **object versioning**
* Configure **lifecycle management rules**
* Validate **VM → Cloud Storage integration**
* Understand **encryption and governance best practices**

---

##  Estimated Time

**90–120 minutes**

---

##  Prerequisites

* Completed **Sessions 1–4 labs**
* Active GCP project with billing enabled
* At least one Compute Engine VM available
* Service account created (from Session 2)
* Cloud Shell enabled

---

##  Lab Scenario (Real-World Context)

You are a **Cloud Infrastructure Engineer** tasked with:

* Storing application logs and artifacts securely
* Ensuring only authorized workloads can write data
* Automatically managing data lifecycle to reduce cost
* Meeting basic security and compliance requirements

---

##  Step 1: Review Cloud Storage Concepts (Checkpoint)

Before starting, confirm understanding of:

* Buckets vs objects
* Storage classes
* IAM vs ACLs
* Versioning and lifecycle rules

Trainer checkpoint 

---

##  Step 2: Create a Cloud Storage Bucket

1. Go to **Cloud Storage → Buckets**
2. Click **Create**
3. Configure:

   * **Bucket name:** `gcp-foundation-storage-<unique-id>`
   * **Location type:** Region
   * **Region:** `asia-south1`
4. Click **Continue**

---

##  Step 3: Configure Storage Class & Access Control

1. Choose **Default storage class:** Standard
2. Enable **Uniform bucket-level access**
3. Disable public access
4. Click **Create**

 **Expected Result:**
A private, regionally located bucket is created

---

##  Step 4: Review Bucket Configuration

1. Open the bucket
2. Review:

   * Location
   * Storage class
   * Public access prevention
   * IAM permissions

💡 **Learning Point:**
Uniform bucket-level access simplifies security

---

##  Step 5: Enable Object Versioning

1. Go to **Bucket → Configuration**
2. Enable **Object versioning**
3. Save changes

 Protects against accidental overwrites or deletions

---

## Step 6: Configure Lifecycle Management Rules

1. Open **Lifecycle** tab
2. Click **Add Rule**
3. Configure rule:

   * **Action:** Set storage class to Coldline
   * **Condition:** Age > 30 days
4. Add second rule:

   * **Action:** Delete object
   * **Condition:** Age > 365 days
5. Save rules

💡 **Learning Point:**
Lifecycle rules automate **cost optimization**

---

## Step 7: Assign IAM Access to Service Account

1. Open **Bucket → Permissions**
2. Click **Grant Access**
3. Add:

   * **Principal:** `vm-storage-sa@PROJECT_ID.iam.gserviceaccount.com`
   * **Role:** Storage Object Creator
4. Save

Service account can upload objects but not delete them

---

## Step 8: Verify No Public Access

1. In bucket settings, confirm:

   * Public access prevention = ON
   * No `allUsers` or `allAuthenticatedUsers` bindings

⚠️ **Security Reminder:**
Never expose buckets publicly unless explicitly required

---

##  Step 9: Upload Objects via Cloud Shell

In **Cloud Shell**, run:

```bash
echo "Hello from GCP Cloud Storage Lab" > sample.txt
gsutil cp sample.txt gs://gcp-foundation-storage-<unique-id>/
```

Verify:

```bash
gsutil ls gs://gcp-foundation-storage-<unique-id>/
```

---

##  Step 10: Test Object Versioning

1. Modify file:

```bash
echo "Updated content" > sample.txt
gsutil cp sample.txt gs://gcp-foundation-storage-<unique-id>/
```

2. List versions:

```bash
gsutil ls -a gs://gcp-foundation-storage-<unique-id>/sample.txt
```

 Multiple versions are displayed

---

##  Step 11: Access Cloud Storage from Compute Engine VM

1. SSH into your VM
2. Run:

```bash
echo "Log from VM" > vm-log.txt
gsutil cp vm-log.txt gs://gcp-foundation-storage-<unique-id>/
```

 **Expected Result:**
Upload succeeds using attached service account (no keys)

---

##  Step 12: Validate Access Boundaries

Try deleting object from VM:

```bash
gsutil rm gs://gcp-foundation-storage-<unique-id>/vm-log.txt
```

❌ **Expected Result:**
Permission denied (least-privilege enforced)

---

##  Step 13: Review Cloud Storage Logs

1. Go to **Cloud Logging → Logs Explorer**
2. Filter:

   * Resource type: `gcs_bucket`
3. Observe:

   * Object creation events
   * Service account identity

💡 **Learning Point:**
Every object operation is auditable

---

##  Step 14: Encryption Overview (Conceptual)

Discuss:

* Default Google-managed encryption
* When CMEK is required
* Role of Cloud KMS in compliance-driven setups

(No CMEK implementation required in this foundation lab)

---

##  Step 15: Review Governance & Cost Controls

Checklist:

* Uniform bucket-level access enabled
* Lifecycle rules configured
* IAM scoped to service account
* No public exposure
* Region aligned with compute

---

##  Lab Validation Checklist

✔ Bucket created securely
✔ Uniform access enabled
✔ Versioning enabled
✔ Lifecycle rules configured
✔ IAM access scoped correctly
✔ VM → bucket integration validated
✔ Unauthorized actions blocked

---

##  Key Takeaways

* Cloud Storage is **object-based**, not file-based
* IAM is the primary security mechanism
* Service accounts eliminate key management
* Lifecycle rules control long-term cost
* Observability and auditability are built-in

---

