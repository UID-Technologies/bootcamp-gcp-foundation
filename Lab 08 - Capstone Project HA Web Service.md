# Lab 8: Capstone Project – High-Availability Web Service

## Lab Overview

This **capstone project** focuses on **high availability and load balancing**—a different pattern from Lab 7. You will build a **Customer-Facing Service Status Portal** that runs on **2 VMs across 2 zones**, fronted by an **HTTP Load Balancer**, so the service stays available even if one VM or zone fails.

| Component | GCP Service | Purpose |
|-----------|-------------|---------|
| **Network** | Custom VPC, Firewall | Isolated network; health-check and HTTP rules |
| **Compute** | 2 VMs (MIG) | Backend servers in different zones |
| **Load Balancing** | HTTP(S) Load Balancer | Distribute traffic; single entry point |
| **Health Checks** | Cloud Monitoring | Ensure only healthy VMs receive traffic |
| **Observability** | Monitoring, Logging | Dashboards and alerts |

---

## Lab 8 vs. Lab 7 – Key Differences

| Aspect | Lab 7 (Internal Dashboard) | Lab 8 (HA Web Service) |
|--------|----------------------------|-------------------------|
| **VMs** | 1 VM | 2 VMs (different zones) |
| **Content** | From Cloud Storage | Baked into VM (startup script) |
| **Entry point** | VM external IP | Load balancer IP |
| **Focus** | Content decoupling, IT updates | High availability, traffic distribution |
| **Services** | Storage + Compute | Load Balancer + MIG + Compute |

---

## Lab Objectives

By the end of this capstone, you will:

- Deploy **2 VMs** in a Managed Instance Group across 2 zones
- Configure an **HTTP Load Balancer** with health checks
- Use a **custom VPC** with firewall rules for load balancer traffic
- Validate **high availability** (stop one VM; service stays up)
- Add **monitoring** for the load-balanced service

---

## Estimated Time

**2.5–3.5 hours**

---

## Prerequisites

This capstone assumes you have completed **Labs 1–6** (and optionally Lab 7). You need:

- A **Google Cloud account** (free trial or paid)
- **Owner** or **Editor** role on a GCP project
- Cloud Shell enabled
- Compute Engine and Load Balancing APIs enabled

---

## Understanding the Use Case

### Business Scenario

Your company needs a **customer-facing service status portal** that:

- Shows "All Systems Operational" or current status to customers
- Must be **highly available**—no single point of failure
- Uses a **single URL** (load balancer IP) so customers don't care which VM serves them
- Survives zone failures—if one zone goes down, the other continues serving

### Architecture Overview

```
                    ┌─────────────────────────────┐
                    │   HTTP Load Balancer         │
                    │   (Single external IP)        │
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │     Backend Service         │
                    │     (Health checks)          │
                    └──────────────┬──────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    │
    ┌─────────────────┐  ┌─────────────────┐          │
    │  VM 1           │  │  VM 2           │          │
    │  Zone A         │  │  Zone B         │          │
    │  (e.g. us-c1-a) │  │  (e.g. us-c1-b)│          │
    └─────────────────┘  └─────────────────┘          │
              │                    │                    │
              └────────────────────┴────────────────────┘
                          Custom VPC
```

### Why This Design?

- **2 VMs in 2 zones** – Zone failure affects only one VM; the other keeps serving
- **Load balancer** – Single IP for customers; traffic distributed across healthy backends
- **Health checks** – Unhealthy VMs are removed from the pool automatically
- **Custom VPC** – Explicit firewall rules; health checks require specific source IPs

---

## Lab Scenario: Real-World Context

You are a **Cloud Engineer** deploying a customer-facing status portal. Your goals:

1. **High availability** – 2 VMs in 2 zones; load balancer in front
2. **Single entry point** – Customers use one URL (load balancer IP)
3. **Automatic failover** – Health checks remove unhealthy VMs from the pool
4. **Secure network** – Custom VPC, minimal firewall rules

---

## Phase 1: Network Foundation

### Step 1.1: Create Custom VPC and Subnet

**Goal:** Create a dedicated VPC for the HA service.

1. Go to **VPC Network → VPC Networks**
2. Click **Create VPC Network**
3. Configure:
   - **Name:** `ha-portal-vpc`
   - **Subnet creation mode:** Custom
4. Add subnet:
   - **Name:** `ha-portal-subnet`
   - **Region:** `us-central1` (or your preferred region)
   - **IP range:** `10.200.1.0/24`
5. Click **Create**

### Step 1.2: Create Firewall Rules

**Goal:** Allow HTTP from the load balancer's health check probes and from the internet.

**Rule 1 – Health checks (required for load balancer):**

1. Go to **VPC Network → Firewall**
2. Click **Create Firewall Rule**
3. Configure:
   - **Name:** `fw-allow-health-check`
   - **Network:** `ha-portal-vpc`
   - **Direction:** Ingress
   - **Source IP ranges:** `130.211.0.0/22` and `35.191.0.0/16` (GCP health check ranges)
   - **Targets:** Specified target tags
   - **Target tags:** `allow-health-check`
   - **Protocols and ports:** TCP: 80
4. Click **Create**

**Rule 2 – SSH (for troubleshooting):**

1. Create:
   - **Name:** `fw-allow-ssh-ha`
   - **Network:** `ha-portal-vpc`
   - **Source:** Your IP or `0.0.0.0/0` (lab only)
   - **Protocols:** TCP: 22
   - **Targets:** All instances (or tag `allow-ssh`)
5. Click **Create**

### Learning Point

Load balancer health checks and proxied traffic come from **specific GCP IP ranges** (`130.211.0.0/22`, `35.191.0.0/16`). You must allow them, or backends will be marked unhealthy and receive no traffic.

---

## Phase 2: Backend VMs (Instance Template + MIG)

### Step 2.1: Create Instance Template

**Goal:** Blueprint for VMs that serve the status page. Each VM will display its own hostname so you can see load balancing in action.

1. Go to **Compute Engine → Instance Templates**
2. Click **Create Instance Template**
3. Configure:
   - **Name:** `ha-portal-template`
   - **Region:** `us-central1`
   - **Machine type:** `e2-micro` (free tier) or `e2-small`
   - **Boot disk:** 10 GB, Debian or Ubuntu
4. **Networking:**
   - **Network:** `ha-portal-vpc`
   - **Subnet:** `ha-portal-subnet`
   - **Network tags:** `allow-health-check`
5. **Management → Automation**, add startup script:

```bash
#!/bin/bash
apt-get update && apt-get install -y apache2
VM_NAME=$(curl -s -H "Metadata-Flavor:Google" http://metadata.google.internal/computeMetadata/v1/instance/name)
ZONE=$(curl -s -H "Metadata-Flavor:Google" http://metadata.google.internal/computeMetadata/v1/instance/zone | cut -d'/' -f4)
echo "<!DOCTYPE html><html><head><title>Service Status</title></head><body>"
echo "<h1>All Systems Operational</h1>"
echo "<p>Served from: <strong>$VM_NAME</strong> (Zone: $ZONE)</p>"
echo "<p>Last checked: $(date -u)</p>"
echo "</body></html>" > /var/www/html/index.html
systemctl enable apache2 && systemctl start apache2
```

6. Click **Create**

### Step 2.2: Create Managed Instance Group (2 VMs, 2 Zones)

**Goal:** Deploy 2 VMs in 2 different zones for high availability.

1. Go to **Compute Engine → Instance Groups**
2. Click **Create Instance Group**
3. Select **New managed instance group (stateless)**
4. Configure:
   - **Name:** `ha-portal-mig`
   - **Location:** Regional (multi-zone)
   - **Region:** `us-central1`
   - **Instance template:** `ha-portal-template`
   - **Autoscaling:** Off
   - **Number of instances:** 2
5. Under **Instance distribution** (if shown), choose **Spread across zones** so one VM goes to zone A and one to zone B
6. Click **Create**

> **Alternative (zonal MIG):** If regional MIG is not available, create a **zonal** MIG in zone `us-central1-a` with 2 instances. For full zone resilience, create a second zonal MIG in `us-central1-b` and add both to the backend service.

### Step 2.3: Add Named Port to Instance Group

**Goal:** Tell the load balancer which port the backends use.

1. Open the instance group `ha-portal-mig`
2. Click **Edit**
3. Under **Port mapping**, click **Add port**
   - **Name:** `http`
   - **Port number:** `80`
4. Click **Save**

### Expected Result

Two VMs running in different zones. Each serves a page showing its hostname and zone.

---

## Phase 3: HTTP Load Balancer

### Step 3.1: Create Health Check

**Goal:** Define what "healthy" means for the load balancer.

1. Go to **Network Services → Load Balancing** (or **Compute Engine → Health Checks**)
2. Click **Create Health Check**
3. Configure:
   - **Name:** `ha-portal-health-check`
   - **Protocol:** HTTP
   - **Port:** 80
   - **Request path:** `/`
4. Click **Create**

### Step 3.2: Create HTTP Load Balancer (Backend + Frontend)

**Goal:** Create the full load balancer with backend service and frontend.

1. Go to **Network Services → Load Balancing** (or **Compute Engine → Load Balancing**)
2. Click **Create Load Balancer**
3. Under **HTTP(S) Load Balancing**, click **Start configuration**
4. Choose **From Internet to my VMs** → **Continue**
5. **Backend configuration:**
   - Click **Backend configuration**
   - **Create or select backend services** → **Create a backend service**
   - **Name:** `ha-portal-backend`
   - **Backend type:** Instance group
   - **Instance group:** `ha-portal-mig`
   - **Port:** 80 (or named port `http`)
   - **Health check:** `ha-portal-health-check` (create if needed)
   - Click **Done**
6. **Host and path rules:** Leave default (all traffic to `ha-portal-backend`)
7. **Frontend configuration:**
   - **Protocol:** HTTP
   - **IP version:** IPv4
   - **IP address:** Ephemeral (or **Create IP address** for a static IP)
   - **Port:** 80
8. Click **Create**

### Step 3.4: Wait for Load Balancer to Be Ready

**Goal:** Allow time for health checks to pass and backends to become healthy.

1. Wait 5–10 minutes
2. Go to **Network Services → Load Balancing**
3. Click on your load balancer
4. Under **Backend**, verify both VMs show as **Healthy**

### Expected Result

Load balancer has an external IP. Both backends are healthy. Traffic is distributed across the 2 VMs.

---

## Phase 4: Validate High Availability

### Step 4.1: Access the Service via Load Balancer IP

**Goal:** Confirm the service is reachable through the load balancer.

1. Note the **Frontend IP** of the load balancer (from the load balancer details page)
2. Open `http://LOAD_BALANCER_IP` in a browser
3. Refresh several times—you should see different VM names (load balancing in action)

### Expected Result

Page shows "All Systems Operational" and either VM's hostname. Refreshing may show the other VM.

### Step 4.2: Simulate Zone Failure (Optional)

**Goal:** Prove that stopping one VM does not take down the service.

1. Go to **Compute Engine → VM Instances**
2. **Stop** one of the MIG instances (the one currently serving may vary)
3. Wait 1–2 minutes for the health check to mark it unhealthy
4. Open `http://LOAD_BALANCER_IP` again—the remaining VM should still serve traffic
5. **Start** the stopped VM to restore full capacity

### Expected Result

Service remains available with one VM down. When both are healthy again, traffic is distributed to both.

### Learning Point

**High availability** means no single point of failure. With 2 VMs in 2 zones, a zone or VM failure does not take down the service.

---

## Phase 5: Observability

### Step 5.1: Create Monitoring Dashboard

**Goal:** View load balancer and backend health in one place.

1. Go to **Monitoring → Dashboards**
2. Create dashboard: `ha-portal-dashboard`
3. Add widgets:
   - **Load balancer request count** (if available)
   - **Backend VM CPU utilization** (filter by instance group)
   - **Health check status** (backend health)

### Step 5.2: Apply Labels

**Goal:** Tag resources for cost and ownership.

1. Edit the instance group `ha-portal-mig` (or each VM)
2. Add labels:
   - `project=ha-portal`
   - `environment=lab`
   - `owner=cloud-team`

### Step 5.3: Explore Logs

**Goal:** See load balancer and VM activity.

1. Go to **Logging → Logs Explorer**
2. Filter: `resource.type="gce_instance"` and `resource.labels.instance_id=~"ha-portal.*"`
3. Or filter for load balancer logs if enabled

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] Custom VPC `ha-portal-vpc` created
- [ ] Firewall rules: health check, HTTP, SSH
- [ ] Instance template `ha-portal-template` with `allow-health-check` tag
- [ ] MIG `ha-portal-mig` with 2 VMs in 2 zones
- [ ] Named port `http:80` on instance group
- [ ] Health check `ha-portal-health-check` created
- [ ] HTTP Load Balancer with backend service
- [ ] Service accessible via load balancer IP
- [ ] (Optional) HA validated—service up with one VM stopped
- [ ] Labels applied

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **High availability** | 2+ VMs in 2+ zones; no single point of failure |
| **Load balancer** | Single entry point; distributes traffic to healthy backends |
| **Health checks** | Unhealthy VMs are removed from the pool automatically |
| **Firewall for LB** | Allow GCP health check IP ranges (`130.211.0.0/22`, `35.191.0.0/16`) |
| **MIG + LB** | Instance group provides the backend pool; template ensures consistency |

---

## Troubleshooting

| Issue | Check |
|------|-------|
| Backends unhealthy | Firewall allows health check IPs; VMs have `allow-health-check` tag; Apache running on port 80 |
| 502 or 503 errors | Wait for health checks to pass (5–10 min); verify both VMs are running |
| Cannot access via LB IP | Firewall allows HTTP (port 80) from 0.0.0.0/0; LB frontend has external IP |
| Only one VM receives traffic | Normal—LB may stick to one backend; refresh or use incognito to see the other |

---

## Lab 7 vs. Lab 8 – When to Use Which

| Scenario | Use Lab 7 Pattern | Use Lab 8 Pattern |
|----------|-------------------|---------------------|
| Content updated by IT without VM access | ✓ (Storage) | |
| Single VM, low traffic | ✓ | |
| Customer-facing, must stay up | | ✓ (HA + LB) |
| 2+ VMs, traffic distribution | | ✓ |
| Zone resilience required | | ✓ |

---

## Next Steps

- Add **HTTPS** with a Google-managed SSL certificate
- Use **Cloud CDN** for static content caching
- Add **Cloud Armor** for DDoS and WAF protection
- Explore **Global external Application Load Balancer** for multi-region HA

---

*Lab 8 – Capstone Project (High-Availability Web Service) | GCP Foundation Bootcamp*
