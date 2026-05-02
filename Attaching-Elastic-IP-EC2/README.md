# 🔗Attaching an Elastic IP to an EC2 Instance

> **Elastic IP = Static IP** — A fixed public IPv4 address allocated to your AWS account that persists across instance stop/start cycles.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Account                          │
│                                                             │
│   ┌──────────────────┐        ┌───────────────────────┐     │
│   │   Elastic IP     │        │     EC2 Instance      │     │
│   │                  │        │   (Elastic-IP-attach  │     │
│   │  3.213.255.201   │◄──────►│                       │     │ 
│   │  (Public Static) │ 1-to-1 │  Private IP:          │     │
│   │                  │ mapping│  172.31.36.111        │     │
│   └──────────────────┘        │  Type: t3.micro       │     │
│                               │  AMI: Ubuntu 26.04 LTS│     │
│                               └───────────────────────┘     │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                   VPC / Subnet                      │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘

          Internet
             │
             ▼
   ┌──────────────────┐
   │  Elastic IP      │  ← Always the same IP, regardless of
   │  3.213.255.201   │    instance stop/start
   └──────────────────┘
             │
             ▼
   ┌──────────────────┐
   │  EC2 Instance    │
   | i-00050d95b31fb7 │
   └──────────────────┘
```

---

## Why Use an Elastic IP?

| Scenario | Without Elastic IP | With Elastic IP |
|---|---|---|
| Instance restarts | Public IP **changes** every time | Public IP **stays the same** |
| DNS / Firewall rules | Must update rules after every restart | Set once, never change |
| Re-association | N/A | Can detach and attach to another instance |

> ⚠️ **Cost Note:** AWS charges for an Elastic IP that is **allocated but not associated** with a running instance. Always release IPs you are not using.

---

## Step-by-Step Guide

### Step 1 — Launch an EC2 Instance

1. Go to **EC2 → Instances → Launch an instance**
2. Set the **Name**: `Elastic-IP-attach`
3. Select **AMI**: Ubuntu Server 26.04 LTS (HVM), SSD Volume Type
4. Choose **Instance type**: `t3.micro` (Free tier eligible)
5. Configure security group and storage as needed (defaults are fine for testing)
6. Click **Launch instance**

---

### Step 2 — Allocate an Elastic IP Address

1. In the EC2 left sidebar, navigate to **Network & Security → Elastic IPs**
2. Click **Allocate Elastic IP address**
3. Under **Public IPv4 address pool**, select **Amazon's pool of IPv4 addresses**
4. Set **Network border group** to your region (e.g., `us-east-1`)
5. Leave all other settings as default
6. Click **Allocate**

Your new Elastic IP (e.g., `3.213.255.201`) will now appear in the list.

---

### Step 3 — Associate the Elastic IP with Your Instance

1. In the **Elastic IP addresses** list, select your newly allocated IP
2. Click **Actions → Associate Elastic IP address**
3. Configure the association:
   - **Resource type**: `Instance`
   - **Instance**: Select your instance (e.g., `i-00050d9aad5b31fb7`)
   - **Private IP address**: Select the instance's private IP (e.g., `172.31.36.111`)
4. Click **Associate**

---

### Step 4 — Verify the Association

1. Go to **EC2 → Instances**
2. Select your instance (`Elastic-IP-attach`)
3. In the **Details** tab, confirm:
   - **Public IPv4 address**: `3.213.255.201` ✅
   - **Private IPv4 address**: `172.31.36.111`
   - **Instance state**: Running

The public IP displayed should now match your allocated Elastic IP and will **not change** after a stop/start.

---

## Managing Your Elastic IP

### Disassociate from an Instance

1. Go to **Elastic IPs**, select your IP
2. Click **Actions → Disassociate Elastic IP address**
3. Confirm the action

The IP remains allocated to your account and can be re-associated with any other instance.

### Re-associate to Another Instance

1. Select the Elastic IP
2. Click **Actions → Associate Elastic IP address**
3. Choose a different instance and associate

### Release the IP (to stop charges)

1. Select the Elastic IP
2. Click **Actions → Release Elastic IP addresses**

> ⚠️ Once released, the IP is returned to the AWS pool and cannot be recovered.

---

## Key Takeaways

- An **Elastic IP is a static public IPv4** address tied to your AWS account, not to any specific instance.
- Without an Elastic IP, your instance's **public IP changes on every restart**.
- You can **move** an Elastic IP between instances instantly — useful for failover scenarios.
- AWS **bills you** when an Elastic IP is allocated but not associated with a running instance.
- Each Elastic IP provides a **1-to-1 mapping** to the primary private IP of your instance within your VPC.


