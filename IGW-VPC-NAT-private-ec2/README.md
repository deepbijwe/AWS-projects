# 🚀AWS VPC — Isolated Network with NAT Gateway

> **Goal:** Build a secure, isolated AWS network where a private EC2 instance has no public IP but can still reach the internet through a NAT Gateway, accessed only via a bastion (jump server) in the public subnet.

---

## Architecture Overview

```
Internet Gateway
       │
       ▼
      VPC (10.0.0.0/16)
       │
 ┌───────────────┬────────────────┐
 │               │                │
 ▼               ▼                ▼
Public Subnet    Private Subnet
(10.0.1.0/24)    (10.0.2.0/24)
 │               │
 │               │
Public EC2       Private EC2
(Bastion Host)   (No Public IP)
 │
 ▼
NAT Gateway (for private subnet internet access)
```

**Traffic flow:**
- Public EC2 ↔ Internet via IGW (direct)
- Private EC2 → Internet via NAT Gateway (outbound only)
- SSH into Private EC2 → must jump through the Public EC2 (bastion host)

---

## Step-by-Step Setup

---

### Step 1 — Create the VPC

1. Navigate to **VPC → Your VPCs → Create VPC**
2. Set the following:
   - **Name tag:** `my-vpc`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **Tenancy:** Default
3. Click **Create VPC**

---

### Step 2 — Create and Attach an Internet Gateway

1. Navigate to **VPC → Internet Gateways → Create Internet Gateway**
2. Set the following:
   - **Name tag:** `my-igw`
3. Click **Create Internet Gateway**
4. Select `my-igw` → **Actions → Attach to VPC**
5. Choose `my-vpc` and click **Attach Internet Gateway**

> An IGW can only be attached to one VPC at a time.

---

### Step 3 — Create the Main Route Table

1. Navigate to **VPC → Route Tables → Create Route Table**
2. Set the following:
   - **Name tag:** `main-rt`
   - **VPC:** `my-vpc`
3. Click **Create Route Table**

---

### Step 4 — Create Public and Private Subnets

Navigate to **VPC → Subnets → Create Subnet**, select `my-vpc`, and add two subnets:

| Subnet | Name | CIDR Block |
|---|---|---|
| Public | `my-public-subnet` | `10.0.1.0/24` |
| Private | `my-private-subnet` | `10.0.105.0/24` |

Create them in the same or different Availability Zones as needed, then click **Create Subnet**.

---

### Step 5 — Configure the Main Route Table for the Public Subnet

**Add internet route:**

1. Select `main-rt` → **Routes tab → Edit Routes → Add Route**
2. Set:
   - **Destination:** `0.0.0.0/0`
   - **Target:** Internet Gateway → select `my-igw`
3. Click **Save Changes**

**Associate the public subnet:**

1. Select `main-rt` → **Subnet Associations tab → Edit Subnet Associations**
2. Check `my-public-subnet`
3. Click **Save Associations**

---

### Step 6 — Create the NAT Gateway

1. Navigate to **VPC → NAT Gateways → Create NAT Gateway**
2. Set the following:
   - **Name:** `my-nat`
   - **Subnet:** `my-public-subnet` *(NAT must live in the public subnet)*
   - **Connectivity type:** Public
   - **Elastic IP:** Click **Allocate Elastic IP**
3. Click **Create NAT Gateway**

> NAT Gateway creation takes **5–10 minutes**. Wait until the status shows **Available** before proceeding.

---

### Step 7 — Create the Private Route Table and Associate It

**Create the route table:**

1. Navigate to **VPC → Route Tables → Create Route Table**
2. Set:
   - **Name tag:** `sub-rt`
   - **VPC:** `my-vpc`
3. Click **Create Route Table**

**Add NAT route:**

1. Select `sub-rt` → **Routes tab → Edit Routes → Add Route**
2. Set:
   - **Destination:** `0.0.0.0/0`
   - **Target:** NAT Gateway → select `my-nat`
3. Click **Save Changes**

**Associate the private subnet:**

1. Select `sub-rt` → **Subnet Associations tab → Edit Subnet Associations**
2. Check `my-private-subnet`
3. Click **Save Associations**

---

### Step 8 — Verify Connections in the VPC Resource Map

1. Navigate to **VPC → Your VPCs → select `my-vpc`**
2. Click the **Resource Map** tab
3. Confirm the following connections are visible:
   - `my-igw` → `my-vpc`
   - `main-rt` → `my-public-subnet` → IGW route
   - `sub-rt` → `my-private-subnet` → NAT route
   - `my-nat` inside the public subnet

---

### Step 9 — Launch EC2 Instances

#### Public EC2 (Bastion Host)

1. Navigate to **EC2 → Launch Instances**
2. Configure:
   - **AMI:** Ubuntu (or preferred)
   - **Instance type:** t2.micro (free tier eligible)
   - **Key pair:** `my-mumbai` (create or select existing)
   - **Network:** `my-vpc`
   - **Subnet:** `my-public-subnet`
   - **Auto-assign Public IP:** Enable
3. Under **Security Group**, allow:
   - **Port 22 (SSH)** from your IP (or `0.0.0.0/0` for testing)
4. Click **Launch Instance**

#### Private EC2

1. Navigate to **EC2 → Launch Instances**
2. Configure:
   - **AMI:** Ubuntu (or preferred)
   - **Instance type:** t2.micro
   - **Key pair:** `my-mumbai` (same key pair)
   - **Network:** `my-vpc`
   - **Subnet:** `my-private-subnet`
   - **Auto-assign Public IP:** Disable (uncheck)
3. Security group — allow SSH from the VPC CIDR (`10.0.0.0/16`) only
4. Click **Launch Instance**

---

### Step 10 — Copy Your Key Pair to the Bastion Host

Before you can SSH into the private instance, you need to copy your `.pem` key to the public (bastion) instance. This enables the SSH jump (also known as a bastion host or jump server pattern).

Open a terminal on your local machine:

```bash
cd ~/Downloads
scp -i my-mumbai.pem my-mumbai.pem ubuntu@15.206.75.28:/home/ubuntu
```

**Expected output:**

```
my-mumbai.pem                                 100% 1678    10.5KB/s   00:00
```

---

### Step 11 — SSH into the Public (Bastion) Instance

```bash
ssh -i my-mumbai.pem ubuntu@15.206.75.28
```

Once connected, verify the key file was copied:

```bash
ls
# my-mumbai.pem
```

---

### Step 12 — Jump to the Private Instance from the Bastion

From inside the bastion (public) instance, restrict the key permissions and SSH into the private instance using its private IP:

```bash
chmod 400 my-mumbai.pem
ssh -i my-mumbai.pem ubuntu@10.0.105.103
```

> This is the **jump server / bastion host** pattern — you access a private, non-routable instance by tunnelling through a publicly accessible instance in the same VPC.

---

### Step 13 — Verify Internet Access from the Private Instance

Once connected to the private EC2, run a ping to verify it can reach the internet through the NAT Gateway:

```bash
ping google.com
```

**Expected output:**

```
PING google.com (142.251.220.46) 56(84) bytes of data.
64 bytes from hkg07s50-in-f14.1e100.net (142.251.220.46): icmp_seq=1 ttl=116 time=1.43 ms
64 bytes from hkg07s50-in-f14.1e100.net (142.251.220.46): icmp_seq=2 ttl=116 time=1.10 ms
64 bytes from hkg07s50-in-f14.1e100.net (142.251.220.46): icmp_seq=3 ttl=116 time=1.11 ms
64 bytes from hkg07s50-in-f14.1e100.net (142.251.220.46): icmp_seq=4 ttl=116 time=1.13 ms
64 bytes from hkg07s50-in-f14.1e100.net (142.251.220.46): icmp_seq=5 ttl=116 time=1.13 ms
64 bytes from hkg07s50-in-f14.1e100.net (142.251.220.46): icmp_seq=6 ttl=116 time=1.14 ms
64 bytes from hkg07s50-in-f14.1e100.net (142.251.220.46): icmp_seq=7 ttl=116 time=1.10 ms
64 bytes from hkg07s50-in-f14.1e100.net (142.251.220.46): icmp_seq=8 ttl=116 time=1.11 ms
...
```

A successful ping confirms:
- The private instance has no public IP
- Internet traffic is routed through the NAT Gateway
- The NAT Gateway is correctly placed in the public subnet with an Elastic IP

---

## Summary

| Resource | Name | Key Config |
|---|---|---|
| VPC | `my-vpc` | `10.0.0.0/16` |
| Internet Gateway | `my-igw` | Attached to `my-vpc` |
| Public Subnet | `my-public-subnet` | `10.0.1.0/24` |
| Private Subnet | `my-private-subnet` | `10.0.105.0/24` |
| Main Route Table | `main-rt` | `0.0.0.0/0 → my-igw` — associated with public subnet |
| Private Route Table | `sub-rt` | `0.0.0.0/0 → my-nat` — associated with private subnet |
| NAT Gateway | `my-nat` | In public subnet, Elastic IP attached |
| Public EC2 | Bastion Host | Public IP, Port 22 open |
| Private EC2 | App / Backend | No public IP, SSH via bastion only |

---

## Key Concepts

**NAT Gateway** allows instances in a private subnet to initiate outbound connections to the internet (e.g., `apt update`, `ping`) without exposing them to inbound internet traffic. The NAT Gateway must reside in the public subnet and have an Elastic IP.

**Bastion Host (Jump Server)** is a hardened public EC2 instance used as the single entry point for SSH access into private instances. You SSH into the bastion first, then SSH again to the private instance from inside the VPC.

**Route Tables** control where network traffic is directed. The public subnet's route table sends `0.0.0.0/0` to the IGW (full internet), while the private subnet's route table sends `0.0.0.0/0` to the NAT Gateway (outbound only).