# 🚀AWS VPC Peering Connection — Step-by-Step Guide
### Mumbai (ap-south-1) VPC-A ↔ N. Virginia (us-east-1) VPC-B

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                            AWS GLOBAL INFRASTRUCTURE                                 │
│                                                                                      │
│  ┌────────────────────────────────────────┐        ┌───────────────────────────────┐ │
│  │     VPC-A  ·  ap-south-1  (Mumbai)     │        │  VPC-B · us-east-1 (Virginia) │ │
│  │     CIDR: 10.0.0.0/16                  │        │  CIDR: 10.1.0.0/16            │ │
│  │                                        │        │                               │ │
│  │  ┌──────────────────────────────────┐  │        │  ┌───────────────────────┐    │ │
│  │  │  Public Subnet: 10.0.1.0/19      │  │        │  │ Private Subnet-B      │    │ │
│  │  │  AZ: ap-south-1a                 │  │        │  │ 10.1.1.0/20           │    │ │
│  │  │                                  │  │        │  │ AZ: us-east-1a        │    │ │
│  │  │  ┌────────────────────────────┐  │  │        │  │                       │    │ │
│  │  │  │   Bastion Host (EC2)       │  │  │        │  │  ┌─────────────────┐  │    │ │
│  │  │  │   Public IP: 15.206.75.28  │  │  │        │  │  │ private-inst-B  │  │    │ │
│  │  │  │   Private IP: 10.0.x.x     │  │  │        │  │  │ 172.16.x.x      │  │    │ │
│  │  │  └────────────┬───────────────┘  │  │        │  │  └────────┬────────┘  │    │ │
│  │  └───────────────│──────────────────┘  │        │  └───────────│───────────┘    │ │
│  │                  │ SSH Jump            │        │             │                 │ │
│  │  ┌───────────────▼──────────────────┐  │        │  ┌──────────▼────────────┐    │ │
│  │  │  Private Subnet: 10.0.2.0/17     │  │        │  │  Private Route Table  │    │ │
│  │  │  AZ: ap-south-1b                 │  │        │  │  10.0.0.0/16 → PCX    │    │ │
│  │  │                                  │  │        │  └───────────────────────┘    | │
│  │  │  ┌────────────────────────────┐  │  │        └───────────────────────────────┘ │
│  │  │  │  private-instance          │  │  │                        ▲                 │
│  │  │  │  IP: 10.0.105.xxx          │  │  │                        │                 │
│  │  │  └────────────────────────────┘  │  │         ┌─────────────┴─────────────┐    │
│  │  └──────────────────────────────────┘  │         │   VPC Peering Connection  │    │
│  │                                        │◄───────►│   pcx-xxxxxxxxxxxxxxxxx   │    │
│  │  ┌──────────────────────────────────┐  │         │  Requester: VPC-A Mumbai  │    │
│  │  │  Internet Gateway:  My-IGW       │  │         │  Accepter:  VPC-B Virginia│    │
│  │  └──────────────────────────────────┘  │         └───────────────────────────┘    │
│  │                                        │                                          │
│  │  ┌──────────────────────────────────┐  │                                          │
│  │  │  Main-RT  (Public Route Table)   │  │                                          │
│  │  │  0.0.0.0/0      → My-IGW         │  │                                          │
│  │  │  10.0.0.0/16    → local          │  │                                          │
│  │  │  172.16.0.0/16  → PCX            │  │                                          │
│  │  └──────────────────────────────────┘  │                                          │
│  │                                        │                                          │
│  │  ┌──────────────────────────────────┐  │                                          │
│  │  │  Private Route Table             │  │                                          │
│  │  │  10.0.0.0/16    → local          │  │                                          │
│  │  │  172.16.0.0/16  → PCX            │  │                                          │
│  │  └──────────────────────────────────┘  │                                          │
│  └────────────────────────────────────────┘                                          │
└──────────────────────────────────────────────────────────────────────────────────────┘

  Traffic Flow:
  Local PC ──SSH──► Bastion Host (public-subnet, VPC-A)
                         └──SSH──► private-instance (private-subnet, VPC-A)
                                        └──PING──► private-inst-B (VPC-B) via PCX
```

---

## Prerequisites

| Item | Detail |
|---|---|
| AWS Account | Active with permissions to create VPCs, EC2, and Peering connections |
| Key Pair | `my-mumbai.pem` created in the Mumbai region |
| Regions | `ap-south-1` (Mumbai) and `us-east-1` (N. Virginia) |
| CIDR Planning | VPC-A: `10.0.0.0/16` · VPC-B: `10.1.0.0/16` — **must not overlap** |

> **Important:** VPC Peering uses private IPs directly between VPCs over the AWS backbone. No NAT Gateway is required.

---

## Part 1 — Build VPC-A in Mumbai (ap-south-1)

### Step 1 — Create the VPC

1. Open the AWS Console → switch region to **Asia Pacific (Mumbai)**.
2. Navigate to **VPC → Your VPCs → Create VPC**.
3. Configure as follows:

| Field | Value |
|---|---|
| Resources to create | VPC only |
| Name tag | `VPC-A` |
| IPv4 CIDR block | `10.0.0.0/16` |
| IPv6 CIDR block | No IPv6 CIDR block |
| Tenancy | Default |

4. Click **Create VPC**. Note the VPC ID — e.g., `vpc-0448ab09bc44e24d5`.

---

### Step 2 — Create an Internet Gateway and Attach It

1. Go to **VPC → Internet Gateways → Create internet gateway**.
2. Name it `My-IGW` → **Create internet gateway**.
3. Select `My-IGW` → **Actions → Attach to VPC**.
4. Choose `VPC-A` → **Attach internet gateway**.

> ✅ IGW state changes from *Detached* → *Attached*.

---

### Step 3 — Create the Public Route Table (Main-RT)

1. Go to **VPC → Route Tables → Create route table**.

| Field | Value |
|---|---|
| Name | `Main-RT` |
| VPC | `VPC-A` |

2. Click **Create route table**.

---

### Step 4 — Add the Internet Route to Main-RT

1. Select `Main-RT` → **Routes tab → Edit routes → Add route**.

| Destination | Target |
|---|---|
| `0.0.0.0/0` | Internet Gateway → `My-IGW` |

2. Click **Save changes**.

> ✅ Main-RT now has two active routes: `10.0.0.0/16 → local` and `0.0.0.0/0 → My-IGW`.

---

### Step 5 — Create Two Subnets

Go to **VPC → Subnets → Create subnet**. Select VPC `my-VPC-custom`, then add both subnets in one flow:

**Subnet 1 — Public**

| Field | Value |
|---|---|
| Subnet name | `public-subnet` |
| Availability Zone | `ap-south-1a` |
| IPv4 subnet CIDR | `10.0.1.0/17` |

**Subnet 2 — Private**

| Field | Value |
|---|---|
| Subnet name | `private-subnet` |
| Availability Zone | `ap-south-1b` |
| IPv4 subnet CIDR | `10.0.2.0/17` |

Click **Create subnet**.

> ✅ Both subnets show as *Available*.

---

### Step 6 — Associate the Public Subnet with Main-RT

1. Select `Main-RT` → **Subnet associations tab → Edit subnet associations**.
2. Check **public-subnet** → **Save associations**.

> The public subnet now routes internet traffic via the IGW. The private subnet stays on the default VPC route table (local only, for now — the peering route will be added later).

---

## Part 2 — Launch EC2 Instances in VPC-A

### Step 7 — Launch the Bastion Host (Public Subnet)

1. Go to **EC2 → Instances → Launch instances**.

| Setting | Value |
|---|---|
| Name | `bastion-host` |
| AMI | Ubuntu 26.04 LTS |
| Instance type | `t3.micro` |
| Key pair | `my-mumbai.pem` |
| VPC | `VPC-A` |
| Subnet | `public-subnet` |
| Auto-assign public IP | **Enable** |
| Security group (inbound) | SSH port 22 from Anywhere, and All ICMP from Anywhere |

2. Click **Launch instance**. Note the **public IP** — e.g., `15.206.75.xx`.

---

### Step 8 — Launch the Private Instance (Private Subnet)

1. Go to **EC2 → Instances → Launch instances**.

| Setting | Value |
|---|---|
| Name | `private-instance` |
| AMI | Ubuntu 26.04 LTS |
| Instance type | `t3.micro` |
| Key pair | `my-mumbai` |
| VPC | `VPC-A` |
| Subnet | `private-subnet` |
| Auto-assign public IP | **Disable** |
| Security group (inbound) | SSH port 22 from anywhere, add ICMP form Anywhere |

2. Click **Launch instance**. Note the **private IP** — e.g., `10.0.105.xxx`.

---

## Part 3 — SSH Bastion Jump into the Private Instance

### Step 9 — Copy the PEM Key to the Bastion Host

From your **local machine**:

```bash
cd Downloads
scp -i my-mumbai.pem my-mumbai.pem ubuntu@15.206.75.28:/home/ubuntu
```

> Type `yes` to accept the host fingerprint. The key file transfers at 100%.

---

### Step 10 — SSH into the Bastion Host

```bash
ssh -i my-mumbai.pem ubuntu@15.206.75.xx
```

---

### Step 11 — SSH from Bastion into the Private Instance

Once logged in to the bastion host:

```bash
# Set correct permissions on the copied key
chmod 400 my-mumbai.pem

# Jump SSH into the private instance using its private IP
ssh -i my-mumbai.pem ubuntu@10.0.105.103
```

> Type `yes` to accept the fingerprint. You are now inside `ubuntu@ip-10-0-105-103`.

---

## Part 4 — Build VPC-B in N. Virginia (us-east-1)

> ⚠️ Switch the AWS Console region to **US East (N. Virginia)** for all steps in this part.

### Step 12 — Create VPC-B

| Field | Value |
|---|---|
| Name tag | `my-VPC-B` |
| IPv4 CIDR | `10.1.0.0/16` |

Click **Create VPC**.

---

### Step 13 — Create a Private Subnet in VPC-B

| Field | Value |
|---|---|
| Subnet name | `private-subnet-B` |
| Availability Zone | `us-east-1a` |
| IPv4 CIDR | `10.1.1.0/20` |

Click **Create subnet**.

---

### Step 14 — Launch a Private EC2 in VPC-B

| Setting | Value |
|---|---|
| Name | `private-instance-B` |
| AMI | Ubuntu 26.04 LTS |
| Instance type | `t3.micro` |
| Key pair | `my-mumbai` (or a Virginia key pair) |
| VPC | `VPC-B` |
| Subnet | `private-subnet` |
| Auto-assign public IP | **Disable** |
| Security group (inbound) | All ICMP – IPv4 from Anywhere |

Note the **private IP** — e.g., `172.16.x.x`.

---

## Part 5 — Create the VPC Peering Connection

### Step 15 — Request the Peering Connection

> Switch back to **Mumbai (ap-south-1)**.

1. Go to **VPC → Peering connections → Create peering connection**.

| Field | Value |
|---|---|
| Name | `VPC-A-to-VPC-B` |
| VPC ID (Requester) | `vpc-0448ab09bc44e24d5` (VPC-A, Mumbai) |
| Account | My account |
| Region | Another region → `us-east-1` |
| VPC ID (Accepter) | VPC-B's ID from N. Virginia |

2. Click **Create peering connection**. Note the Peering Connection ID — e.g., `pcx-xxxxxxxxxxxxxxxxx`.

---

### Step 16 — Accept the Peering Connection

> Switch the console to **N. Virginia (us-east-1)**.

1. Go to **VPC → Peering connections**.
2. Select the connection in *Pending acceptance* state.
3. **Actions → Accept request → Accept request**.

> ✅ Status changes to *Active*.

---

## Part 6 — Update Route Tables for Peering Traffic

> ⚠️ This is the most critical part. Routes must be added on **both sides**. Incorrect or missing CIDRs here are the #1 cause of peering failures.

### Step 17 — Update VPC-A Route Tables (Mumbai)

> Switch back to **Mumbai (ap-south-1)**.

**Main-RT (public subnet route table):**

Go to **Route Tables → Main-RT → Routes → Edit routes → Add route**:

| Destination | Target |
|---|---|
| `10.1.0.0/16` | Peering Connection → `pcx-xxxxxxxxxxxxxxxxx` |

**Private subnet route table (default VPC route table):**

Go to **Route Tables → select the route table associated with private-subnet → Routes → Edit routes → Add route**:

| Destination | Target |
|---|---|
| `10.1.0.0/16` | Peering Connection → `pcx-xxxxxxxxxxxxxxxxx` |

Click **Save changes** on both.

---

### Step 18 — Update VPC-B Route Table (N. Virginia)

> Switch to **N. Virginia (us-east-1)**.

Select the route table associated with `private-subnet-B` → **Routes → Edit routes → Add route**:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | Peering Connection → `pcx-xxxxxxxxxxxxxxxxx` |

Click **Save changes**.

---

## Part 7 — Verify Cross-VPC Connectivity

### Step 19 — Ping VPC-B Private Instance from VPC-A

From inside `private-instance` (VPC-A, `10.0.105.103`):

```bash
ping 172.16.x.x
```

**Expected output:**

```
PING 172.16.x.x (172.16.x.x) 56(84) bytes of data.
64 bytes from 172.16.x.x: icmp_seq=1 ttl=116 time=1.43 ms
64 bytes from 172.16.x.x: icmp_seq=2 ttl=116 time=1.10 ms
64 bytes from 172.16.x.x: icmp_seq=3 ttl=116 time=1.11 ms
64 bytes from 172.16.x.x: icmp_seq=4 ttl=116 time=1.13 ms
```

> ✅ Successful ping confirms VPC Peering is fully working between Mumbai and N. Virginia.

---

## Troubleshooting Checklist

| Symptom | Likely Cause | Fix |
|---|---|---|
| Ping times out | Route missing in VPC-A route table | Add `172.16.0.0/16 → PCX` in both VPC-A route tables |
| Ping times out | Route missing in VPC-B route table | Add `10.0.0.0/16 → PCX` in VPC-B route table |
| Ping times out | **Wrong CIDR in route** (e.g., `/17` instead of `/16`) | Correct destination to the full VPC CIDR — this was the actual fix in this lab |
| Ping times out | VPC-B security group blocking ICMP | Add inbound: All ICMP – IPv4 from `10.0.0.0/16` |
| Peering stuck in *Pending* | Accepter region not switched | Switch console to `us-east-1` and accept the request |
| SSH to private instance fails | PEM file permissions too open | Run `chmod 400 my-mumbai.pem` on the bastion host |
| SCP fails | Wrong key or public IP | Confirm bastion's public IP and port 22 is open in its security group |

---


## Key Concepts

- **VPC Peering** is a private, non-transitive network link between two VPCs. Traffic travels over the AWS backbone and never hits the public internet.
- **No NAT Gateway needed** — instances communicate directly using their private IPs via the peering connection.
- **Routes must be added on both sides** — peering does not auto-populate route tables. Each VPC must explicitly route to the peer's CIDR via the PCX ID.
- **Security groups must allow the traffic** — the receiving instance's SG must permit ICMP (for ping) or port 22 (for SSH) from the peer VPC's CIDR.
- **CIDRs must not overlap** — peering cannot be established between VPCs with overlapping IP ranges.
- **Peering is non-transitive** — if VPC-A peers with VPC-B and VPC-B peers with VPC-C, VPC-A cannot reach VPC-C through VPC-B. Each pair needs its own peering connection.

---


 Docuement Author :- Deep bijwe 