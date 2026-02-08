#  Lab 3: VPC Networking – Step-by-Step

##  Lab Objectives

By the end of this lab, participants will:

* Design and create a **custom VPC** in Google Cloud Platform
* Create **regional subnets** with proper CIDR planning
* Configure **firewall rules** for secure access
* Understand **routes and internet connectivity**
* Validate **private vs public communication** using Compute Engine VMs

---

##  Estimated Time

**90–120 minutes**

---

##  Prerequisites

* Completed **Session 1 & Session 2 labs**
* Active GCP project with billing enabled
* Owner or Network Admin role
* Cloud Shell enabled

---

##  Lab Scenario (Real-World Context)

You are a **Cloud Infrastructure Engineer** tasked with:

* Creating a secure network for application workloads
* Separating environments using IP ranges
* Allowing controlled SSH access
* Enabling internet access while minimizing exposure

---

![Image](https://docs.cloud.google.com/static/architecture/images/vpc-bps-native-firewall-rules.svg)

![Image](https://d33wubrfki0l68.cloudfront.net/57750035ef2c7221ce9bdece3154ff503d5fdc9c/a0271/gcpimages/02-architecture/00-public-private-subnet.png)

![Image](https://docs.cloud.google.com/static/architecture/images/hybrid-multicloud-secure-networking-patterns/gated-e-i.svg)

![Image](https://docs.cloud.google.com/static/firewall/images/firewall-policies/firewall-policy-tutorial.svg)

![Image](https://user-images.githubusercontent.com/59575502/191422505-a0025cb9-ef2d-48ec-a5c4-4ce5f20c4f38.png)

---

##  Step 1: Review Default VPC (Baseline Understanding)

1. Open **GCP Console**
2. Navigate to:
   **VPC Network → VPC Networks**
3. Click on **default** VPC
4. Observe:

   * Auto-created subnets
   * Predefined firewall rules

 **Learning Point:**
Default VPC is convenient but **not recommended for production**

---

##  Step 2: Create a Custom VPC

1. Go to **VPC Network → VPC Networks**
2. Click **Create VPC Network**
3. Configure:

   * **Name:** `custom-app-vpc`
   * **Subnet creation mode:** Custom
4. Disable default firewall rules (recommended)
5. Click **Create**

 **Expected Result:**
A clean, enterprise-ready custom VPC is created

---

##  Step 3: Create Regional Subnets

Create two subnets:

### Subnet 1 – Application Subnet

* **Name:** `app-subnet-asia`
* **Region:** `asia-south1`
* **IP range:** `10.10.1.0/24`

### Subnet 2 – Future Expansion

* **Name:** `app-subnet-us`
* **Region:** `us-central1`
* **IP range:** `10.20.1.0/24`

Click **Done** → **Create**

 **CIDR Planning Tip:**
Always leave space for future growth

---

##  Step 4: Review Routes

1. Navigate to **VPC Network → Routes**
2. Filter by network: `custom-app-vpc`
3. Observe:

   * Default local subnet routes
   * Default internet gateway route (`0.0.0.0/0`)

 **Learning Point:**
Routes are auto-created but critical for connectivity

---

##  Step 5: Create Firewall Rule for SSH Access

1. Go to **VPC Network → Firewall**
2. Click **Create Firewall Rule**
3. Configure:

   * **Name:** `allow-ssh-admin`
   * **Network:** `custom-app-vpc`
   * **Direction:** Ingress
   * **Source IP ranges:** `YOUR_PUBLIC_IP/32` *(or 0.0.0.0/0 for lab only)*
   * **Protocols/Ports:** TCP:22
   * **Target:** All instances (or via tag `ssh-access`)
4. Click **Create**

⚠️ **Security Note:**
Never allow `0.0.0.0/0` in production

---

##  Step 6: Create Firewall Rule for Internal Traffic

1. Click **Create Firewall Rule**
2. Configure:

   * **Name:** `allow-internal`
   * **Network:** `custom-app-vpc`
   * **Direction:** Ingress
   * **Source IP ranges:** `10.10.0.0/16`
   * **Protocols:** All
3. Click **Create**

 Enables **internal VM-to-VM communication**

---

##  Step 7: Launch a VM in Custom VPC

1. Navigate to **Compute Engine → VM Instances**
2. Click **Create Instance**
3. Configure:

   * **Name:** `app-vm-1`
   * **Region:** `asia-south1`
   * **Zone:** `asia-south1-a`
   * **Network:** `custom-app-vpc`
   * **Subnet:** `app-subnet-asia`
   * **External IP:** Enabled
4. Click **Create**

---

##  Step 8: Verify VM Networking

1. Click on `app-vm-1`
2. Note:

   * Internal IP (10.10.x.x)
   * External IP
3. SSH into VM via Console

Inside VM, run:

```bash
ip a
ping -c 3 google.com
```

 **Expected Result:**
VM has internet connectivity

---

##  Step 9: Validate Firewall Rules

1. Temporarily remove SSH firewall rule
2. Attempt SSH again
3. Observe failure
4. Re-enable firewall rule

💡 **Learning Point:**
Firewall rules are **enforced immediately**

---

##  Step 10: Private vs Public IP Discussion

Discuss:

* Why internal IPs are safer
* When public IPs are required
* How Cloud NAT is used for outbound-only access (conceptual)

---

##  Step 11: Validate Networking via CLI

In Cloud Shell:

```bash
gcloud compute networks list
gcloud compute networks subnets list
gcloud compute firewall-rules list --filter=custom-app-vpc
```

---

##  Step 12: Clean Architecture Review

Final review checklist:

* Custom VPC used
* CIDR ranges planned
* Firewall rules minimal
* VM deployed securely

---

##  Lab Validation Checklist

✔ Custom VPC created
✔ Subnets created in multiple regions
✔ Firewall rules configured
✔ VM launched in custom subnet
✔ Internet connectivity validated
✔ CLI and Console both used

---

##  Key Takeaways

* VPCs are **global**
* Subnets are **regional**
* Firewall rules are stateful
* CIDR planning is critical
* Custom VPCs are mandatory for production

---
