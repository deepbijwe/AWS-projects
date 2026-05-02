# Attaching a Network Interface to an EC2 Instance
> Allocate a private IP for internal communication using AWS Elastic Network Interfaces (ENI)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        AWS Region (us-east-1)                   │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   VPC (Default)                          │   │
│  │                                                          │   │
│  │  ┌──────────────────────────────────────────────────|    │   │
│  │  │              Subnet (us-east-1d)                 |    │   │
│  │  │                                                  |    │   │
│  │  │   ┌──────────────────────────────────────────┐   │    │   │
│  │  │   │         EC2 Instance (t3.micro)          │   │    │   │
│  │  │   │         Name: Network-interface          │   │    │   │
│  │  │   │                                          │   │    │   │
│  │  │   │  ┌─────────────────────────────────────┐ │   │    │   │
│  │  │   │  │   Primary ENI (eth0)                │ │   │    │   │
│  │  │   │  │   Private IP : 172.31.36.111        │ │   │    │   │
│  │  │   │  │   Public IP  : 54.165.153.84        │ │   │    │   │
│  │  │   │  │   SG: launch-wizard-1, default      │ │   │    │   │
│  │  │   │  └─────────────────────────────────────┘ │   │    │   │
│  │  │   │                                          │   │    │   │
│  │  │   │  ┌─────────────────────────────────────┐ │   │    │   │
│  │  │   │  │   Secondary ENI (eth1) ← NEW        │ │   │    │   │
│  │  │   │  │   ENI ID : eni-064e256a931f5164a    │ │   │    │   │
│  │  │   │  │   Private IP : 172.31.37.161        │ │   │    │   │
│  │  │   │  │   Description: network-interface    │ │   │    │   │
│  │  │   │  │   SG: launch-wizard-1, default      │ │   │    │   │
│  │  │   │  └─────────────────────────────────────┘ │   │    │   │
│  │  │   └──────────────────────────────────────────┘   │    │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

        Internal traffic flows between both private IPs within the VPC.
        The secondary ENI can be detached and reattached to other
        instances within the same VPC and subnet.
```

---

## Prerequisites

| Requirement | Details |
|---|---|
| AWS Account | Active account with EC2 permissions |
| EC2 Instance | A running instance (used as attachment target) |
| Network Interface Service | Access to EC2 > Network Interfaces in the console |

---

## Step 1 — Launch an EC2 Instance

1. Navigate to **EC2 → Instances → Launch instances**.
2. Configure the instance:
   - **Name:** `Network-interface`
   - **AMI:** Ubuntu (latest LTS)
   - **Key pair:** Optional for this walkthrough
   - **Network:** Default VPC
3. Click **Launch instance**.
4. Once running, **note the Availability Zone (AZ)** of your instance — you will need it in the next step. In this example the instance is in `us-east-1d`.

---

## Step 2 — Create a Network Interface

1. In the EC2 console left sidebar, go to **Network & Security → Network Interfaces**.
2. Click **Create network interface**.
3. Fill in the form:

   | Field | Value |
   |---|---|
   | Description | `network-interface` (optional label) |
   | Subnet | Choose the subnet that matches your instance's AZ |
   | Interface type | ENA (default) |
   | Private IPv4 address | Auto-assign |
   | Security groups | Select the same security group(s) as your instance |

4. Click **Create network interface**.
   - ✅ A success banner will confirm: *"Successfully created network interface eni-064e256a931f5164a"*

> **Important:** The subnet must be in the **same AZ** as your EC2 instance. A mismatch will prevent attachment.

---

## Step 3 — Attach the Network Interface to Your Instance

1. On the Network Interfaces list, select the newly created ENI.
2. Click **Actions → Attach**.
3. In the **Attach network interface** dialog:
   - **VPC:** Select the default VPC (`vpc-03d48d297c0354076`)
   - **Instance:** Select your EC2 instance (`i-00050d9aad5b31fb7 / Network-interface`)
4. Click **Attach**.
   - ✅ A success banner will confirm: *"Successfully attached network interface eni-064e256a931f5164a to instance i-00050d9aad5b31fb7"*

> **Note:** The t3.micro instance type does not support ENA Express or ENA queues — this is expected and does not affect basic ENI attachment.

---

## Step 4 — Verify the Attachment

1. Navigate to **EC2 → Instances** and select your instance.
2. In the **Details** tab, scroll to **Private IPv4 addresses**.
3. You should now see **two private IP addresses** assigned to the instance:

   | Interface | Private IP | Public IP |
   |---|---|---|
   | Primary ENI (eth0) | `172.31.36.111` | `54.165.153.84` |
   | Secondary ENI (eth1) | `172.31.37.161` | — |

The secondary private IP enables **internal VPC communication** — other resources can now route traffic to this instance using either private IP.

---

## Key Concepts & Best Practices

**When to use a secondary ENI:**
- Host multiple services on a single instance, each reachable by a distinct private IP.
- Migrate a network interface (and its IP) from one instance to another with zero downtime.
- Separate management traffic from application traffic.

**Detaching and reattaching:**
- You can **detach** the ENI from one instance and **attach** it to another.
- Both instances must be in the **same VPC and same subnet**.

**Cost warning ⚠️**
> AWS charges for Elastic IP addresses that are **allocated but not associated** with a running instance. Always release unattached Elastic IPs and delete unused ENIs to avoid unexpected charges.

---

## Summary of Resources Created

| Resource | ID / Value |
|---|---|
| EC2 Instance | `i-00050d9aad5b31fb7` |
| Primary ENI | Auto-created with instance |
| Secondary ENI | `eni-064e256a931f5164a` |
| Subnet | `subnet-05eeff679442a94fe` |
| VPC | `vpc-03d48d297c0354076` |
| Security Groups | `sg-007860de920dcfb71` (launch-wizard-1), `sg-030ec6a29ae96cae0` (default) |
| Region | `us-east-1` (N. Virginia) |