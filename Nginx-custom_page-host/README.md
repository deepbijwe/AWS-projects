

# 🚀 Hosting a Static Web Page on Nginx (AWS EC2 - Ubuntu)

This documentation demonstrates how to deploy a simple static web page using **Nginx** on an **AWS EC2 Ubuntu instance**.

---

## 📌 Prerequisites

* AWS account
* Basic knowledge of EC2
* SSH client (or AWS EC2 Instance Connect)

---

## 🧭 Step 1: Launch an EC2 Instance

1. Navigate to **AWS Console → EC2 → Launch Instance**
2. Configure the instance:

* **Name**: `nginx-host`
* **AMI (OS)**: Ubuntu
* **Instance Type**: `t2.micro` (Free Tier eligible)
* **Key Pair**: Create or select an existing key pair
* **Network Settings**:

  * Allow **HTTP (Port 80)**
  * Allow **SSH (Port 22)**

---

### 🖼️ EC2 Launch Configuration (Example)

---

## ⚙️ Step 2: Configure Using User Data (Automated Setup)

In **Advanced Details → User Data**, paste the following script:

```bash
#!/bin/bash
sudo -i
apt update -y
apt install nginx -y

echo "this is nginx custom page" > index.html
cp index.html /var/www/html

systemctl reload nginx
```

### 🔍 What This Script Does

* Updates package lists
* Installs Nginx
* Creates a custom HTML page
* Copies it to `/var/www/html`
* Reloads the Nginx service

---

## 🔐 Alternative: Manual Setup via SSH

If you did not use User Data, follow these steps:

### Connect to EC2

```bash
ssh -i your-key.pem ubuntu@<your-public-ip>
```

### Install and Configure Nginx

```bash
sudo apt update -y
sudo apt install nginx -y

echo "this is nginx custom page" > index.html
sudo cp index.html /var/www/html/

sudo systemctl reload nginx
```

---

## 🌐 Step 3: Access Your Web Page

1. Copy your **EC2 Public IP**
   Example: `13.233.230.125`

2. Open your browser and visit:

```
http://<your-public-ip>
```

---

### 🖼️ Expected Browser Output

---

## ✅ Expected Result

You should see:

```
this is nginx custom page
```

---

## 🏗️ Architecture Diagram

### 📌 High-Level Flow

```
+-------------------+
|   End User        |
| (Web Browser)     |
+---------+---------+
          |
          | HTTP Request (Port 80)
          ▼
+--------------------------+
|   AWS Internet Gateway   |
+------------+-------------+
             |
             ▼
+--------------------------+
|   EC2 Instance           |
|   (Ubuntu OS)            |
|                          |
|   +------------------+   |
|   |   Nginx Server   |   |
|   |  (Port 80 Open)  |   |
|   +--------+---------+   |
|            |             |
|            ▼             |
|   /var/www/html          |
|   (index.html file)      |
+--------------------------+
             |
             ▼
      HTTP Response
   ("this is nginx custom page")
```

---

### 🧭 Architecture Explanation

* User sends a request via browser using EC2 public IP
* Request passes through AWS Internet Gateway
* Reaches EC2 instance (Ubuntu)
* Nginx handles HTTP request on port 80
* Static content served from `/var/www/html`
* Response returned to user

---

### 🖼️ Conceptual Architecture View

---

## 🎉 Conclusion

You have successfully:

* Launched an EC2 instance
* Installed and configured Nginx
* Hosted a static web page
* Accessed it via a public IP

---

## 🧠 Best Practices & Improvements

* Use **Elastic IP** for a fixed public IP
* Restrict SSH access to your IP only
* Add a **Domain Name (Route 53)**
* Enable **HTTPS using Let’s Encrypt (Certbot)**
* Use **Application Load Balancer (ALB)** for scaling

---

## 📚 Future Enhancements (For Resume/Projects)

* Infrastructure as Code using **Terraform**
* CI/CD pipeline using **GitHub Actions**
* Dockerize Nginx setup
* Monitoring with **CloudWatch**

---

