# Lab 7: Capstone Project – Internal Resource Dashboard

## Lab Overview

This **capstone project** brings together skills from Labs 1–6 into a single, real-world solution. You will build an **Internal Company Resource Dashboard**—a web application that displays company announcements and links, with content managed via Cloud Storage so IT can update it without touching servers.

| Component | GCP Service | Purpose |
|-----------|-------------|---------|
| **Network** | VPC, Firewall | Secure, isolated network |
| **Compute** | Compute Engine | Web server hosting the dashboard |
| **Content** | Cloud Storage | Store and update content (no VM access needed) |
| **Identity** | IAM, Service Accounts | Least-privilege access |
| **Observability** | Monitoring, Logging | Dashboards and alerts |
| **Governance** | Labels | Cost tracking and ownership |

---

## Lab Objectives

By the end of this capstone, you will:

- Design and deploy a **multi-service solution** on GCP
- Integrate **Compute Engine** with **Cloud Storage** for content delivery
- Apply **IAM** and **VPC** best practices from earlier labs
- Add **monitoring, alerts, and labels** for production readiness
- Validate the end-to-end flow and troubleshoot if needed

---

## Estimated Time

**2–3 hours**

---

## Prerequisites

This capstone assumes you have completed **Labs 1–6** or have equivalent knowledge. You need:

- A **Google Cloud account** (free trial or paid)
- **Owner** or **Editor** role on a GCP project
- Cloud Shell enabled
- Compute Engine, Cloud Storage, and Monitoring APIs enabled

---

## Understanding the Use Case

### Business Scenario

Your company needs an **internal resource dashboard** that:

- Shows announcements, links, and quick references for employees
- Can be updated by IT **without SSH access** to servers—they simply upload a new file to Cloud Storage
- Runs securely with proper network isolation and least-privilege access
- Is monitored so issues are detected early

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         GCP Project                               │
│  ┌─────────────┐    ┌──────────────┐    ┌─────────────────────┐ │
│  │   VPC       │    │ Cloud        │    │  Compute Engine VM   │ │
│  │   +         │───▶│ Storage      │◀───│  (Web Server)        │ │
│  │   Firewall  │    │ (Content)    │    │  - Fetches content   │ │
│  └─────────────┘    └──────────────┘    │  - Serves dashboard  │ │
│         │                   │           └─────────────────────┘ │
│         │                   │                     │              │
│         │                   │                     ▼              │
│         │                   │           ┌─────────────────────┐ │
│         │                   │           │ Monitoring & Logging │ │
│         │                   └──────────▶│ Alerts, Dashboards   │ │
│         │                               └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Why This Design?

- **Cloud Storage for content** – IT uploads a JSON file; the VM fetches and displays it. No code deployment for content updates.
- **Service account** – VM uses attached identity to read from the bucket; no keys to manage.
- **Custom VPC** – Network isolation and explicit firewall rules.
- **Monitoring & labels** – Production-ready observability and cost governance.

---

## Lab Scenario: Real-World Context

You are a **Cloud Engineer** tasked with deploying the internal resource dashboard. Your goals:

1. **Deploy securely** – VPC, firewall, least-privilege IAM
2. **Enable easy updates** – Content in Cloud Storage; IT uploads, VM serves
3. **Make it observable** – Dashboard, alerts, logs
4. **Apply governance** – Labels for cost and ownership

---

## Phase 1: Foundation (Network & Identity)

### Step 1.1: Create or Reuse a Custom VPC

**Goal:** Ensure you have a custom VPC with subnets.

**Option A – Create new (if you don't have one):**

1. Go to **VPC Network → VPC Networks**
2. Click **Create VPC Network**
3. **Name:** `capstone-vpc`
4. **Subnet creation mode:** Custom
5. Add subnet:
   - **Name:** `capstone-subnet`
   - **Region:** `us-central1` (or your preferred region)
   - **IP range:** `10.100.1.0/24`
6. Click **Create**

**Option B – Reuse** `custom-app-vpc` from Lab 3 if it exists.

### Step 1.2: Create Firewall Rules

**Goal:** Allow HTTP (port 80) and SSH (port 22).

1. Go to **VPC Network → Firewall**
2. Create **allow-http-dashboard**:
   - **Network:** `capstone-vpc` (or your VPC)
   - **Direction:** Ingress
   - **Source:** `0.0.0.0/0` (for lab; restrict in production)
   - **Protocols:** TCP: 80
3. Create **allow-ssh-capstone**:
   - **Network:** `capstone-vpc`
   - **Direction:** Ingress
   - **Source:** Your IP or `0.0.0.0/0` (lab only)
   - **Protocols:** TCP: 22

### Step 1.3: Create Service Account for the VM

**Goal:** Identity for the VM with read-only access to the content bucket.

1. Go to **IAM & Admin → Service Accounts**
2. Click **Create Service Account**
3. **Name:** `dashboard-vm-sa`
4. **Roles:** `Storage Object Viewer` (at project level, or we'll scope to bucket)
5. Click **Done**
6. **Do not create keys** – we will use attached identity

---

## Phase 2: Content Storage

### Step 2.1: Create Cloud Storage Bucket

**Goal:** Bucket to store dashboard content.

1. Go to **Cloud Storage → Buckets**
2. Click **Create**
3. **Name:** `capstone-dashboard-content-YOUR_PROJECT_ID` (globally unique)
4. **Location:** Same region as your VM (e.g., `us-central1`)
5. **Access control:** Uniform
6. **Public access prevention:** Enforced
7. Click **Create**

### Step 2.2: Grant Service Account Access to Bucket

**Goal:** Allow the VM's service account to read objects.

1. Open the bucket → **Permissions** tab
2. Click **Grant Access**
3. **Principal:** `dashboard-vm-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com`
4. **Role:** `Storage Object Viewer`
5. Click **Save**

### Step 2.3: Upload Dashboard Content

**Goal:** Create the JSON content file that the dashboard will display.

1. In Cloud Shell, create a file:

```bash
cat > dashboard-content.json << 'EOF'
{
  "title": "Company Resource Dashboard",
  "updated": "2024-01-15",
  "announcements": [
    {
      "date": "2024-01-15",
      "message": "Welcome to the internal resource dashboard. Bookmark this page!"
    }
  ],
  "links": [
    {"name": "IT Helpdesk", "url": "https://helpdesk.example.com"},
    {"name": "HR Portal", "url": "https://hr.example.com"},
    {"name": "GCP Console", "url": "https://console.cloud.google.com"}
  ]
}
EOF
```

2. Upload to the bucket:

```bash
gsutil cp dashboard-content.json gs://YOUR_BUCKET_NAME/content.json
```

Replace `YOUR_BUCKET_NAME` with your bucket name.

---

## Phase 3: Compute (Web Server)

### Step 3.1: Create VM with Startup Script

**Goal:** Deploy a VM that fetches content from Cloud Storage and serves a simple dashboard.

1. Go to **Compute Engine → VM Instances**
2. Click **Create Instance**
3. Configure:
   - **Name:** `dashboard-vm`
   - **Region/Zone:** Same as subnet (e.g., `us-central1-a`)
   - **Machine type:** `e2-micro` (free tier) or `e2-small`
   - **Boot disk:** 10 GB, Debian or Ubuntu
4. **Networking:**
   - **Network:** `capstone-vpc`
   - **Subnet:** `capstone-subnet`
   - **External IP:** Ephemeral
5. **Identity:**
   - **Service account:** `dashboard-vm-sa`
6. **Labels:** Add `project=capstone`, `environment=lab`
7. Under **Management → Automation**, add this startup script.

**Replace `YOUR_BUCKET_NAME`** with your actual bucket name (e.g., `capstone-dashboard-content-myproject`):

```bash
#!/bin/bash
BUCKET="YOUR_BUCKET_NAME"
apt-get update && apt-get install -y nginx jq

# Fetch content and generate HTML
gsutil cp gs://$BUCKET/content.json /tmp/content.json 2>/dev/null || true
if [ -f /tmp/content.json ]; then
  TITLE=$(jq -r '.title' /tmp/content.json)
  UPDATED=$(jq -r '.updated' /tmp/content.json)
  echo "<!DOCTYPE html><html><head><title>$TITLE</title></head><body>"
  echo "<h1>$TITLE</h1><p>Last updated: $UPDATED</p>"
  echo "<h2>Announcements</h2><ul>"
  jq -r '.announcements[]? | "<li>\(.date): \(.message)</li>"' /tmp/content.json 2>/dev/null
  echo "</ul><h2>Quick Links</h2><ul>"
  jq -r '.links[]? | "<li><a href=\"\(.url)\">\(.name)</a></li>"' /tmp/content.json 2>/dev/null
  echo "</ul></body></html>" > /var/www/html/index.html
else
  echo "<html><body><h1>Dashboard</h1><p>Content not yet available. Check bucket: $BUCKET</p></body></html>" > /var/www/html/index.html
fi

systemctl enable nginx && systemctl start nginx
```

8. Click **Create**

### Step 3.2: How Content Updates Work

The startup script fetches content **once at boot**. To see updated content after uploading a new `content.json`:

- **Option A:** Restart the VM (Compute Engine → VM Instances → Select `dashboard-vm` → Restart). The startup script runs again and fetches fresh content.
- **Option B:** SSH in and manually run: `gsutil cp gs://YOUR_BUCKET/content.json /tmp/content.json` then regenerate the HTML (or add a cron job for auto-refresh).

For this capstone, **Option A** is sufficient. The key learning: content lives in Storage; IT updates the file, and the VM serves it—no code deployment needed.

### Step 3.3: Verify the Dashboard

1. Wait 2–3 minutes for the VM to start and the script to complete
2. Note the **External IP** of `dashboard-vm`
3. Open `http://EXTERNAL_IP` in a browser

**Expected Result:** A page showing the dashboard content (title, announcements, links or JSON).

---

## Phase 4: Observability

### Step 4.1: Create a Monitoring Dashboard

**Goal:** Single view of dashboard VM health.

1. Go to **Monitoring → Dashboards**
2. Click **Create Dashboard**
3. Name: `capstone-dashboard`
4. Add widgets:
   - **CPU utilization** – Resource: VM Instance, Metric: CPU utilization, Filter: `dashboard-vm`
   - **Network traffic** – Metric: Network bytes received
5. Save

### Step 4.2: Create an Alert Policy

**Goal:** Get notified if CPU is high.

1. Go to **Monitoring → Alerting**
2. **Create Policy**
3. Condition: CPU utilization > 70% for 2 minutes, Target: `dashboard-vm`
4. Notification: Your email
5. Name: `capstone-high-cpu-alert`
6. Create

### Step 4.3: Apply Labels (Governance)

**Goal:** Tag resources for cost tracking.

1. Edit `dashboard-vm` → add labels:
   - `project=capstone`
   - `environment=lab`
   - `owner=cloud-team`
2. Edit the Cloud Storage bucket → **Configuration** tab → add same labels
3. Save

### Step 4.4: Explore Logs

**Goal:** See VM and bucket activity in Cloud Logging.

1. Go to **Logging → Logs Explorer**
2. Filter: `resource.type="gce_instance"` and `resource.labels.instance_id="YOUR_INSTANCE_ID"`
3. Or filter: `resource.type="gcs_bucket"` to see bucket access

---

## Phase 5: Content Update (Validate the Flow)

### Step 5.1: Update Content Without Touching the VM

**Goal:** Prove that IT can update the dashboard by uploading a new file.

1. Edit `dashboard-content.json` locally or in Cloud Shell:

```bash
cat > dashboard-content.json << 'EOF'
{
  "title": "Company Resource Dashboard",
  "updated": "2024-01-16",
  "announcements": [
    {"date": "2024-01-16", "message": "New: IT will perform maintenance tonight 10 PM - 11 PM."},
    {"date": "2024-01-15", "message": "Welcome to the internal resource dashboard!"}
  ],
  "links": [
    {"name": "IT Helpdesk", "url": "https://helpdesk.example.com"},
    {"name": "HR Portal", "url": "https://hr.example.com"},
    {"name": "GCP Console", "url": "https://console.cloud.google.com"}
  ]
}
EOF
```

2. Upload:

```bash
gsutil cp dashboard-content.json gs://YOUR_BUCKET_NAME/content.json
```

3. **Restart the VM** (Compute Engine → VM Instances → Select `dashboard-vm` → Restart) so the startup script runs again and fetches the new content.
4. Wait 2–3 minutes, then refresh the browser – new content should appear.

**Expected Result:** Dashboard shows the updated announcement without any VM configuration changes.

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] Custom VPC and firewall rules created
- [ ] Service account `dashboard-vm-sa` created (no keys)
- [ ] Cloud Storage bucket created with `content.json`
- [ ] VM `dashboard-vm` deployed with correct service account and startup script
- [ ] Dashboard accessible at `http://VM_EXTERNAL_IP`
- [ ] Monitoring dashboard created
- [ ] Alert policy configured
- [ ] Labels applied to VM
- [ ] Content update flow validated (upload new JSON → see changes)

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **Multi-service integration** | VPC + Compute + Storage + IAM work together. Design for security and simplicity. |
| **Content in Storage** | Decouple content from compute. Update content without redeploying. |
| **Service accounts** | Attached identity > keys. VM reads from bucket with no credentials on disk. |
| **Observability** | Dashboards, alerts, and labels make the solution production-ready. |
| **End-to-end validation** | Always test the full flow, including content updates. |

---

## Troubleshooting

| Issue | Check |
|------|-------|
| Dashboard shows "502" or blank | Startup script may still be running. Wait 3–5 min. SSH and check `systemctl status nginx`. |
| "Permission denied" when VM fetches | Service account has Storage Object Viewer on the bucket. Verify bucket name in script. |
| Cannot access via HTTP | Firewall allows TCP 80. VM has external IP. |
| Content not updating | Cron runs every 5 min. Or restart VM after upload. |

---

## Next Steps

- Add **Cloud Load Balancing** to distribute traffic across multiple VMs
- Use **Cloud CDN** for faster content delivery
- Add **Cloud Armor** for DDoS protection
- Explore **Terraform** or **Deployment Manager** to automate this deployment

---

*Lab 7 – Capstone Project | GCP Foundation Bootcamp*
