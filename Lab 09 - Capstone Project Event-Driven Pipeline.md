# Lab 9: Capstone Project – Event-Driven Document Intake Pipeline

## Lab Overview

This **capstone project** introduces **serverless, event-driven architecture**—a different pattern from Labs 7 and 8. You will build an **Automated Document Intake System** that processes files the moment they are uploaded to Cloud Storage, without any VMs or manual intervention.

| Component | GCP Service | Purpose |
|-----------|-------------|---------|
| **Storage** | Cloud Storage (2 buckets) | Inbox for uploads; archive for processed files |
| **Compute** | Cloud Functions | Serverless processing—runs only when triggered |
| **Events** | Eventarc (Storage trigger) | Automatically invokes function on file upload |
| **Identity** | IAM, Service Accounts | Least-privilege access for the function |
| **Observability** | Logging, Monitoring | Trace processing and alert on failures |

---

## Lab 9 vs. Labs 7 & 8 – Key Differences

| Aspect | Lab 7 | Lab 8 | Lab 9 |
|--------|-------|-------|-------|
| **Compute** | 1 VM (always on) | 2 VMs (always on) | Cloud Functions (on-demand) |
| **Trigger** | Manual / VM boot | User request | File upload (event) |
| **Pattern** | Content decoupling | High availability | Event-driven, serverless |
| **Cost model** | Pay for VM 24/7 | Pay for 2 VMs 24/7 | Pay per invocation |
| **Services** | Storage, Compute, VPC | Load Balancer, MIG | Functions, Storage, Eventarc |

---

## Lab Objectives

By the end of this capstone, you will:

- Deploy a **Cloud Function** triggered by Cloud Storage events
- Configure **Eventarc** to connect Storage events to the function
- Use **IAM** with a dedicated function service account
- Build a **multi-bucket workflow** (inbox → processed/rejected)
- Add **monitoring and logging** for the pipeline
- Validate the **end-to-end event-driven flow**

---

## Estimated Time

**2–2.5 hours**

---

## Prerequisites

This capstone assumes you have completed **Labs 1–6** (and optionally Labs 7–8). You need:

- A **Google Cloud account** (free trial or paid)
- **Owner** or **Editor** role on a GCP project
- Cloud Shell enabled
- Cloud Storage, Cloud Functions, Eventarc, and Cloud Build APIs enabled

---

## Understanding the Use Case

### Business Scenario

Your company receives documents (invoices, reports, forms) that employees or partners upload to a shared location. You need a system that:

- **Processes files automatically** when they are uploaded—no manual steps
- **Validates file types** (e.g., allow PDF, TXT, JSON; reject others)
- **Organizes output**—valid files go to an archive; invalid files go to a "rejected" folder
- **Leaves an audit trail**—every processing action is logged
- **Runs without servers**—no VMs to manage; pay only when files are processed

### Architecture Overview

```
    User uploads file
            │
            ▼
┌─────────────────────────┐
│  Bucket: document-inbox │  ◀── Trigger: object finalized
└───────────┬─────────────┘
            │
            │  Eventarc (Storage trigger)
            ▼
┌─────────────────────────┐
│  Cloud Function         │
│  - Validate file type   │
│  - Copy to processed/   │
│    rejected             │
│  - Log result           │
└───────────┬─────────────┘
            │
            ├──────────────────┬──────────────────┐
            ▼                  ▼                  ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ document-archive│  │ document-inbox  │  │ Cloud Logging    │
│ /processed/     │  │ /rejected/      │  │ (audit trail)    │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### Why This Design?

- **Event-driven** – Function runs only when a file is uploaded; no idle compute cost
- **Serverless** – No VMs to patch, scale, or monitor for uptime
- **Separation of concerns** – Inbox for raw uploads; archive for processed; rejected for invalid files
- **Auditability** – Every action logged to Cloud Logging

---

## Lab Scenario: Real-World Context

You are a **Cloud Engineer** building an automated document intake pipeline. Your goals:

1. **Automate processing** – No manual file handling
2. **Enforce validation** – Only allowed file types are archived
3. **Use serverless** – Minimize cost and operational overhead
4. **Ensure observability** – Logs and metrics for troubleshooting

---

## Phase 1: Enable APIs and Create Buckets

### Step 1.1: Enable Required APIs

**Goal:** Activate Cloud Functions, Eventarc, and Cloud Build.

1. Open **Cloud Shell**
2. Run:

```bash
gcloud services enable cloudfunctions.googleapis.com
gcloud services enable eventarc.googleapis.com
gcloud services enable cloudbuild.googleapis.com
gcloud services enable storage.googleapis.com
```

3. Verify:

```bash
gcloud services list --enabled | grep -E "cloudfunctions|eventarc|cloudbuild"
```

### Expected Result

APIs are enabled. Cloud Functions (2nd gen) uses Eventarc for triggers.

### Step 1.2: Create Storage Buckets

**Goal:** Create an inbox bucket (for uploads) and an archive bucket (for processed files).

1. Set your project and region:

```bash
export PROJECT_ID=$(gcloud config get-value project)
export REGION=us-central1
```

2. Create buckets:

```bash
# Inbox - where users upload files
gsutil mb -l $REGION gs://${PROJECT_ID}-doc-inbox

# Archive - where processed files are stored
gsutil mb -l $REGION gs://${PROJECT_ID}-doc-archive
```

3. Enable uniform bucket-level access (if not default):

```bash
gsutil uniformbucketlevelaccess set on gs://${PROJECT_ID}-doc-inbox
gsutil uniformbucketlevelaccess set on gs://${PROJECT_ID}-doc-archive
```

### Expected Result

Two buckets exist. Bucket names are globally unique; using project ID helps avoid conflicts.

### Learning Point

**Inbox vs. archive** – Separating upload location from processed output keeps the pipeline clear and allows different lifecycle rules per bucket.

---

## Phase 2: Create the Cloud Function

### Step 2.1: Create Function Directory and Code

**Goal:** Prepare the function source code.

1. In Cloud Shell:

```bash
mkdir -p ~/doc-processor-function
cd ~/doc-processor-function
```

2. Create `main.py`:

```python
import functions_framework
from google.cloud import storage
import logging
import os

# Allowed file extensions
ALLOWED_EXTENSIONS = {'.pdf', '.txt', '.json', '.csv'}

@functions_framework.event
def process_document(event, context):
    """Triggered by Cloud Storage when a file is uploaded to the inbox bucket."""
    bucket_name = event.get('bucket')
    file_name = event.get('name')
    if not bucket_name or not file_name:
        logging.error("Missing bucket or file name in event")
        return
    
    # Skip if file is in processed/ or rejected/ (avoid re-processing)
    if file_name.startswith('processed/') or file_name.startswith('rejected/'):
        logging.info(f"Skipping {file_name} - already processed")
        return
    
    storage_client = storage.Client()
    source_bucket = storage_client.bucket(bucket_name)
    source_blob = source_bucket.blob(file_name)
    
    # Get file extension
    ext = os.path.splitext(file_name)[1].lower()
    
    if ext in ALLOWED_EXTENSIONS:
        # Valid: copy to archive bucket under processed/
        dest_bucket_name = os.environ.get('ARCHIVE_BUCKET', f'{os.environ.get("GCP_PROJECT", "unknown")}-doc-archive')
        dest_bucket = storage_client.bucket(dest_bucket_name)
        dest_blob = dest_bucket.blob(f'processed/{file_name}')
        source_bucket.copy_blob(source_blob, dest_bucket, dest_blob.name)
        # Delete from inbox after successful copy
        source_blob.delete()
        logging.info(f"Processed: {file_name} -> archive/processed/{file_name}")
    else:
        # Invalid: move to rejected folder in same bucket
        dest_blob = source_bucket.blob(f'rejected/{file_name}')
        source_bucket.copy_blob(source_blob, source_bucket, dest_blob.name)
        source_blob.delete()
        logging.warning(f"Rejected: {file_name} (invalid type {ext}) -> inbox/rejected/{file_name}")
```

3. Create `requirements.txt`:

```
functions-framework
google-cloud-storage
```

### Step 2.2: Deploy the Cloud Function

**Goal:** Deploy the function with a Storage trigger on the inbox bucket.

> **Console alternative:** Go to **Cloud Functions** → **Create Function**. Choose **2nd gen**, set trigger to **Cloud Storage**, select the inbox bucket, and paste the code. Set environment variables `ARCHIVE_BUCKET` and `GCP_PROJECT`.

1. Get your project ID and set the archive bucket:

```bash
export PROJECT_ID=$(gcloud config get-value project)
export REGION=us-central1
export INBOX_BUCKET=${PROJECT_ID}-doc-inbox
export ARCHIVE_BUCKET=${PROJECT_ID}-doc-archive
```

2. Deploy (2nd gen, Storage trigger):

```bash
gcloud functions deploy doc-processor \
  --gen2 \
  --runtime=python312 \
  --region=$REGION \
  --source=. \
  --entry-point=process_document \
  --trigger-bucket=$INBOX_BUCKET \
  --set-env-vars="ARCHIVE_BUCKET=$ARCHIVE_BUCKET,GCP_PROJECT=$PROJECT_ID" \
  --memory=256MB \
  --timeout=60s
```

3. Wait 3–5 minutes for deployment to complete.

### Expected Result

Function `doc-processor` is deployed. It triggers when a file is uploaded to the inbox bucket.

### Learning Point

**Storage trigger** – The function runs automatically when an object is created in the specified bucket. No polling or manual invocation needed.

### Note on Event Format

With `--trigger-bucket`, the function receives a **legacy event**—a dict with `bucket` and `name` keys. The `@functions_framework.event` decorator is used for this format.

---

## Phase 3: IAM and Permissions

### Step 3.1: Verify Function Service Account Permissions

**Goal:** Ensure the function can read from the inbox and write to the archive.

1. The default Compute Engine service account or the function's service account needs:
   - `roles/storage.objectViewer` on the inbox bucket (read, delete)
   - `roles/storage.objectCreator` on the archive bucket (write)
   - `roles/storage.objectAdmin` on the inbox bucket (copy within bucket for rejected)

2. Get the function's service account:

```bash
gcloud functions describe doc-processor --region=$REGION --gen2 --format="value(serviceConfig.serviceAccountEmail)"
```

3. Grant permissions (replace `FUNCTION_SA` with the output above, or use default):

```bash
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
export FUNCTION_SA=${PROJECT_NUMBER}-compute@developer.gserviceaccount.com

# Inbox: read, delete, copy (for rejected)
gsutil iam ch serviceAccount:${FUNCTION_SA}:objectViewer gs://${INBOX_BUCKET}
gsutil iam ch serviceAccount:${FUNCTION_SA}:objectAdmin gs://${INBOX_BUCKET}

# Archive: write
gsutil iam ch serviceAccount:${FUNCTION_SA}:objectCreator gs://${ARCHIVE_BUCKET}
```

### Expected Result

The function can read from the inbox, write to the archive, and move files within the inbox (rejected folder).

### Learning Point

**Least privilege** – Grant only the permissions the function needs. ObjectAdmin on inbox allows copy and delete; ObjectCreator on archive allows writes.

---

## Phase 4: Test the Pipeline

### Step 4.1: Upload a Valid File

**Goal:** Trigger the function with an allowed file type.

1. Create and upload a test file:

```bash
echo "Test document content" > test-doc.txt
gsutil cp test-doc.txt gs://${INBOX_BUCKET}/
```

2. Wait 10–30 seconds.

3. Check the archive bucket:

```bash
gsutil ls gs://${ARCHIVE_BUCKET}/processed/
```

4. Check that the file was removed from the inbox:

```bash
gsutil ls gs://${INBOX_BUCKET}/
```

### Expected Result

- `test-doc.txt` appears in `gs://ARCHIVE_BUCKET/processed/`
- `test-doc.txt` is removed from the inbox root

### Step 4.2: Upload an Invalid File

**Goal:** Verify rejected files go to the rejected folder.

1. Create and upload a disallowed file:

```bash
echo "Invalid type" > test-doc.exe
gsutil cp test-doc.exe gs://${INBOX_BUCKET}/
```

2. Wait 10–30 seconds.

3. Check the rejected folder:

```bash
gsutil ls gs://${INBOX_BUCKET}/rejected/
```

### Expected Result

- `test-doc.exe` appears in `gs://INBOX_BUCKET/rejected/`
- `test-doc.exe` is removed from the inbox root

### Step 4.3: View Logs

**Goal:** Confirm processing is logged.

1. Go to **Logging → Logs Explorer**
2. Filter:

```
resource.type="cloud_run_revision"
resource.labels.service_name="doc-processor"
```

3. Or filter by severity: `severity>=INFO`

### Expected Result

Log entries showing "Processed" or "Rejected" for each file.

### Learning Point

**Cloud Logging** captures all function output. Use it for debugging and audit trails.

---

## Phase 5: Observability

### Step 5.1: Create a Monitoring Dashboard

**Goal:** View function invocations and errors.

1. Go to **Monitoring → Dashboards**
2. Create dashboard: `doc-processor-dashboard`
3. Add widgets:
   - **Cloud Run** (or Cloud Functions) → **Request count** for `doc-processor`
   - **Cloud Run** → **Error rate** (if available)
   - **Cloud Run** → **Request latency**

### Step 5.2: Create an Alert for Failures

**Goal:** Get notified when the function fails.

1. Go to **Monitoring → Alerting**
2. **Create Policy**
3. Condition: **Cloud Run** or **Cloud Functions** → **Error rate** > 0 (or use logs-based metric for "ERROR" severity)
4. Notification: Your email
5. Name: `doc-processor-failures`
6. Create

### Step 5.3: Apply Labels (Optional)

**Goal:** Tag resources for cost and ownership.

1. Labels can be added when deploying. Add to the deploy command:

```bash
--update-labels=project=doc-pipeline,environment=lab
```

2. Or add labels to the Storage buckets in the Console: **Bucket → Configuration → Labels**.

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] APIs enabled (Cloud Functions, Eventarc, Cloud Build)
- [ ] Buckets created: `doc-inbox` and `doc-archive`
- [ ] Cloud Function `doc-processor` deployed with Storage trigger
- [ ] IAM permissions granted for function service account
- [ ] Valid file (.txt, .pdf, .json, .csv) → moved to archive/processed/
- [ ] Invalid file (.exe, etc.) → moved to inbox/rejected/
- [ ] Logs visible in Cloud Logging
- [ ] Monitoring dashboard created (optional)
- [ ] Alert configured for failures (optional)

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **Event-driven** | Functions run only when events occur; no idle cost |
| **Serverless** | No VMs to manage; automatic scaling |
| **Storage trigger** | Eventarc connects Storage events to Functions |
| **IAM** | Function uses a service account; grant minimal permissions |
| **Separation** | Inbox vs. archive vs. rejected keeps the pipeline clear |

---

## Troubleshooting

| Issue | Check |
|------|-------|
| Function not triggering | Trigger points to correct bucket; Eventarc API enabled; check Logging for errors |
| Permission denied | Function SA has objectViewer/objectAdmin on inbox, objectCreator on archive |
| Wrong event format | Gen2 uses CloudEvent; ensure `@functions_framework.cloud_event` and correct payload parsing |
| File not moving | Verify bucket names in env vars; check logs for exceptions |

---

## Alternative: Eventarc (CloudEvent) format

If you prefer Eventarc with `--trigger-event-filters`, the function receives a CloudEvent. Use `@functions_framework.cloud_event` and access `cloud_event.data.bucket` and `cloud_event.data.name`. The event format may differ; refer to [Cloud Functions Eventarc docs](https://cloud.google.com/functions/docs/calling/eventarc).

---

## Lab 7, 8, 9 – When to Use Which

| Scenario | Use Lab 7 | Use Lab 8 | Use Lab 9 |
|----------|-----------|-----------|-----------|
| Content updated by IT, no VM access | ✓ | | |
| Customer-facing, must stay up | | ✓ | |
| File upload triggers processing | | | ✓ |
| Minimize cost (pay per use) | | | ✓ |
| No servers to manage | | | ✓ |
| Need load balancing | | ✓ | |

---

## Next Steps

- Add **Pub/Sub** for async notifications when processing completes
- Use **Cloud Scheduler** to trigger a function for batch processing
- Add **Cloud Storage lifecycle rules** to move old archive files to Coldline
- Explore **Cloud Workflows** for multi-step pipelines

---

*Lab 9 – Capstone Project (Event-Driven Document Intake Pipeline) | GCP Foundation Bootcamp*
