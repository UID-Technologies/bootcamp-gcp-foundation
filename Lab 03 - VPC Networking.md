# Lab 3: VPC Networking

## Lab Overview

This **standalone lab** teaches you how to design and secure networks in Google Cloud Platform (GCP). You will build a custom network with proper isolation and access controls.

| Pillar | What It Does | Why It Matters |
|--------|--------------|----------------|
| **VPC** | Virtual Private Cloud—your isolated network in GCP | Controls connectivity and segmentation |
| **Subnets** | IP ranges within regions for your resources | Enables CIDR planning and regional isolation |
| **Firewall Rules** | Allow or deny traffic to/from instances | Enforces security at the network boundary |

---

## Lab Objectives

By the end of this lab, you will:

- Design and create a **custom VPC** in Google Cloud Platform
- Create **regional subnets** with proper CIDR planning
- Configure **firewall rules** for secure access
- Understand **routes and internet connectivity**
- Validate **private vs public communication** using Compute Engine VMs

---

## Estimated Time

**90–120 minutes**

---

## Prerequisites (Standalone)

This lab is designed to run **on its own**. You need:

- A **Google Cloud account** (free trial or paid)
- **Owner** or **Network Admin** role on a GCP project
- Cloud Shell enabled
- Compute Engine API enabled

**If you don't have a project yet:** Complete Lab 1 or create a project, link billing, and enable Compute Engine API.

---

## Understanding the Concepts

### What is a VPC?

A **Virtual Private Cloud (VPC)** is your private network in GCP. It is **global**—it spans all regions. Resources in the same VPC can communicate (subject to firewall rules) even across regions.

**Example:** A VM in `us-central1` and a VM in `asia-south1` can talk over private IPs if they share a VPC.

### What are Subnets?

**Subnets** are regional IP ranges within your VPC. Each subnet has a CIDR block (e.g., `10.10.1.0/24`). VMs get their internal IP from the subnet of the zone they run in.

**Example:** `app-subnet-asia` with `10.10.1.0/24` in `asia-south1` gives you 254 usable IPs in that region.

### What are Firewall Rules?

Firewall rules control **ingress** (incoming) and **egress** (outgoing) traffic. They are **stateful**—if you allow a connection in, the response is automatically allowed. Default VPC has permissive rules; custom VPCs start with deny-all (except for some default allows).

**Example:** A rule allowing TCP port 22 from `YOUR_IP/32` enables SSH only from your machine.

---

## Lab Scenario: Real-World Context

You are a **Cloud Infrastructure Engineer** tasked with:

1. **Creating a secure network** for application workloads
2. **Separating environments** using IP ranges (e.g., dev vs. prod)
3. **Allowing controlled SSH access** from your IP only
4. **Enabling internet access** for VMs while minimizing exposure

This lab walks you through building that network.

---

## Quick Setup (If You Don't Have a Project)

**Skip this section if you already have a GCP project.**

1. Create a project (Lab 1, Step 4)
2. Link billing
3. Enable Compute Engine API
4. Proceed to Step 1

---

## Step 1: Review Default VPC (Baseline Understanding)

**Goal:** Understand what the default VPC provides and why custom VPCs are preferred.

### Steps

1. Open **GCP Console**
2. Navigate to **VPC Network → VPC Networks**
3. Click on the **default** VPC
4. Observe:
   - Auto-created subnets (one per region)
   - Predefined firewall rules (e.g., `default-allow-ssh`, `default-allow-internal`)

### Expected Result

You see the default network structure. It is convenient for quick tests but not recommended for production.

### Learning Point

The default VPC is **permissive** and shared. For production, use a **custom VPC** with explicit firewall rules and CIDR planning.

---

## Step 2: Create a Custom VPC

**Goal:** Create a clean, enterprise-ready VPC.

### Steps

1. Go to **VPC Network → VPC Networks**
2. Click **Create VPC Network**
3. Configure:
   - **Name:** `custom-app-vpc`
   - **Subnet creation mode:** Custom
4. Leave subnets empty for now (we will add them in the next step)
5. Under **Firewall**, you can disable "Create firewall rules" if you want full control (optional—for this lab, we will create our own rules)
6. Click **Create**

### Expected Result

A new VPC `custom-app-vpc` appears. It has no subnets yet.

### Learning Point

Custom VPCs give you **full control** over subnets and firewall rules. Start with a clean slate.

---

## Step 3: Create Regional Subnets

**Goal:** Add subnets with proper CIDR planning for growth.

### Steps

1. Click on **custom-app-vpc** to edit it
2. Click **Add Subnet**
3. Create **Subnet 1 – Application (Asia)**:
   - **Name:** `app-subnet-asia`
   - **Region:** `asia-south1`
   - **IP range:** `10.10.1.0/24`
4. Click **Add Subnet** again
5. Create **Subnet 2 – Future Expansion (US)**:
   - **Name:** `app-subnet-us`
   - **Region:** `us-central1`
   - **IP range:** `10.20.1.0/24`
6. Click **Save**

### Expected Result

Two subnets in different regions. CIDR ranges do not overlap (`10.10.x` vs. `10.20.x`).

### Learning Point

**CIDR planning** matters. Use different ranges per region or environment. Leave space for future growth (e.g., `/24` = 254 IPs; use `/23` or larger if you expect more).

---

## Step 4: Review Routes

**Goal:** Understand how traffic is routed in your VPC.

### Steps

1. Navigate to **VPC Network → Routes**
2. Filter by network: `custom-app-vpc`
3. Observe:
   - **Local routes** – One per subnet (traffic stays within VPC)
   - **Default route** (`0.0.0.0/0`) – Sends traffic to the internet gateway for VMs with external IPs

### Expected Result

Routes show how traffic flows. The default route enables internet access for VMs with public IPs.

### Learning Point

Routes are **auto-created** for subnets. The default route is critical for outbound internet. For private-only VMs, use Cloud NAT instead of external IPs.

---

## Step 5: Create Firewall Rule for SSH Access

**Goal:** Allow SSH only from your IP (or a controlled range).

### Steps

1. Go to **VPC Network → Firewall**
2. Click **Create Firewall Rule**
3. Configure:
   - **Name:** `allow-ssh-admin`
   - **Network:** `custom-app-vpc`
   - **Direction:** Ingress
   - **Source IP ranges:** `YOUR_PUBLIC_IP/32`
     - Find your IP: [https://whatismyip.com](https://whatismyip.com) or run `curl ifconfig.me` in Cloud Shell
     - For lab only, you can use `0.0.0.0/0` (not for production)
   - **Protocols and ports:** TCP: 22
   - **Target:** All instances in the network (or use tag `ssh-access` for more control)
4. Click **Create**

### Expected Result

A new firewall rule. SSH is allowed only from the specified source.

### Learning Point

**Never use `0.0.0.0/0` for SSH in production.** Restrict to your office IP or VPN range. Firewall rules are enforced immediately.

---

## Step 6: Create Firewall Rule for Internal Traffic

**Goal:** Allow all traffic between VMs within the VPC.

### Steps

1. Click **Create Firewall Rule**
2. Configure:
   - **Name:** `allow-internal`
   - **Network:** `custom-app-vpc`
   - **Direction:** Ingress
   - **Source IP ranges:** `10.10.0.0/16` (covers both `10.10.x` and `10.20.x` subnets)
   - **Protocols and ports:** All
   - **Target:** All instances
3. Click **Create**

### Expected Result

VMs in the VPC can communicate with each other over private IPs.

### Learning Point

Internal rules use **private CIDR ranges** as source. This enables VM-to-VM communication (e.g., app server to database).

---

## Step 7: Launch a VM in Custom VPC

**Goal:** Deploy a VM in your custom subnet.

### Steps

1. Navigate to **Compute Engine → VM Instances**
2. Click **Create Instance**
3. Configure:
   - **Name:** `app-vm-1`
   - **Region:** `asia-south1`
   - **Zone:** `asia-south1-a`
   - **Network:** `custom-app-vpc`
   - **Subnet:** `app-subnet-asia`
   - **External IP:** Ephemeral (default, for SSH access)
4. Click **Create**

### Expected Result

A VM running in `app-subnet-asia` with an internal IP (10.10.1.x) and external IP.

### Learning Point

VMs get their internal IP from the subnet. External IP is optional—use it only when you need direct inbound access (e.g., SSH). For outbound-only, use Cloud NAT.

---

## Step 8: Verify VM Networking

**Goal:** Confirm the VM has network connectivity.

### Steps

1. Click on `app-vm-1`
2. Note the **Internal IP** (10.10.x.x) and **External IP**
3. Click **SSH** to open a browser-based terminal
4. Inside the VM, run:

```bash
ip a
ping -c 3 google.com
```

### Expected Result

- `ip a` shows the internal IP on the primary interface
- `ping` succeeds—VM has outbound internet access

### Learning Point

The default route and external IP enable internet access. Without firewall rules, inbound traffic (except SSH if allowed) is blocked.

---

## Step 9: Validate Firewall Rules (Optional)

**Goal:** See that firewall rules are enforced immediately.

### Steps

1. Temporarily **delete** or disable the `allow-ssh-admin` rule
2. Try to SSH into the VM again—it should fail
3. **Recreate** the `allow-ssh-admin` rule
4. SSH should work again

### Expected Result

Firewall changes take effect immediately. No VM restart needed.

### Learning Point

Firewall rules are **stateful** and **immediate**. Test rules in a non-production environment first.

---

## Step 10: Private vs Public IP Discussion (Conceptual)

**Goal:** Understand when to use private vs. public IPs.

### Summary

| Use Case | Approach |
|----------|----------|
| **Inbound SSH** | External IP + firewall rule (or IAP Tunnel) |
| **Web server** | Load balancer with external IP; VMs can be private |
| **Outbound-only (e.g., updates)** | No external IP; use Cloud NAT |
| **Internal-only (e.g., database)** | Private IP only; no external IP |

### Learning Point

**Internal IPs are safer**—they are not reachable from the internet. Use public IPs only when necessary. Cloud NAT provides outbound internet for private VMs.

---

## Step 11: Validate Networking via CLI

**Goal:** Use `gcloud` to list networks and firewall rules.

### Steps

1. In Cloud Shell, run:

```bash
gcloud compute networks list
gcloud compute networks subnets list
gcloud compute firewall-rules list --filter="network:custom-app-vpc"
```

### Expected Result

Tables showing your VPC, subnets, and firewall rules.

### Learning Point

CLI is useful for **automation** and **scripting**. Same resources, different interface.

---

## Step 12: Clean Architecture Review

**Goal:** Confirm your setup meets best practices.

### Checklist

- [ ] Custom VPC used (not default)
- [ ] CIDR ranges planned (no overlap)
- [ ] Firewall rules minimal (SSH + internal only)
- [ ] VM deployed in correct subnet
- [ ] No overly permissive rules (e.g., 0.0.0.0/0 for all ports)

### Learning Point

A **clean architecture** reduces attack surface and simplifies troubleshooting. Document your design for future reference.

---

## Lab Validation Checklist

Before finishing, confirm:

- [ ] Custom VPC `custom-app-vpc` created
- [ ] Subnets created in multiple regions
- [ ] Firewall rules configured (SSH + internal)
- [ ] VM `app-vm-1` launched in custom subnet
- [ ] Internet connectivity validated (ping)
- [ ] CLI and Console both used

---

## Key Takeaways

| Concept | Takeaway |
|---------|----------|
| **VPC** | Global, isolated network. Subnets are regional. |
| **Subnets** | Regional IP ranges. Plan CIDR for growth. |
| **Firewall rules** | Stateful, immediate. Deny by default; allow explicitly. |
| **CIDR planning** | Critical for scalability. Avoid overlapping ranges. |
| **Custom VPC** | Mandatory for production. Default is for quick tests only. |

---

## Next Steps

- **Lab 4** – Compute Engine & Virtual Machines (uses this VPC)
- Explore **Cloud NAT** for private VMs needing outbound internet
- Explore **VPC Peering** for connecting VPCs

---

*Lab 3 – VPC Networking | GCP Foundation Bootcamp*
