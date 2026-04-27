
## 🔐 Step 1: Create Security Group (EFS-SG)

1. Navigate to **EC2 → Security Groups → Create Security Group**.
2. Configure:
   - **Name:** `EFS-SG`
   - **Inbound Rules:**
     | Type | Port | Source |
     |------|------|--------|
     | SSH  | 22   | My IP  |
     | NFS  | 2049 | My IP  |
3. Click **Create Security Group**.

---

## 🖥️ Step 2: Launch EC2 Instances

### Instance 1: NFS-server-01
- **AMI:** Amazon Linux
- **Name:** `NFS-server-01`
- **Availability Zone:** `us-south-1a`
- **Public IP:** Enabled
- **Security Group:** `EFS-SG`
- **Key Pair:** Select or create

### Instance 2: NFS-server-02
- **AMI:** Amazon Linux
- **Name:** `NFS-server-02`
- **Availability Zone:** `us-south-1b`
- **Public IP:** Enabled
- **Security Group:** `EFS-SG`
- **Key Pair:** Select or create

---

## 📂 Step 3: Create Amazon EFS

1. Navigate to **AWS Console → EFS → Create File System**.
2. **Name:** `MY-EFS`
3. Disable **Automatic Backups** (optional).
4. Add **Mount Targets**:
   - **AZ 1a:** Default VPC with `EFS-SG`
   - **AZ 1b:** Default VPC with `EFS-SG`
5. Click **Create** and wait 2–5 minutes for the mount targets to become available.

---

## 🔑 Step 4: Connect to EC2 Instances

```bash
ssh -i <your-key.pem> ec2-user@<public-ip>
````

Repeat for both instances.

---

## 📦 Step 5: Install Amazon EFS Utilities

Run the following commands on **both instances**:

```bash
sudo yum update -y
sudo yum install -y amazon-efs-utils
```

---

## 📁 Step 6: Create Mount Directory

```bash
mkdir data
ls
```

---

## 🔗 Step 7: Mount the EFS File System

1. Go to **EFS Console → Attach** and copy the **DNS mount command**:

   ```bash
   sudo mount -t efs -o tls fs-xxxxxxxx:/ /home/ec2-user/data
   ```

2. Replace `fs-xxxxxxxx` with your actual **File System ID**.

3. Run the command on **both instances**.

---

## ✅ Step 8: Verify the Mount

```bash
df -hT
```

**Expected Output Example:**

```
127.0.0.1:/   8.0E   0   8.0E   0%   /root/data
```

---

## 🔄 Step 9: Verify Shared Storage

### On `NFS-server-01`

```bash
cd data
touch file{1..10}
ls
```

### On `NFS-server-02`

```bash
cd data
ls
```

**Expected Output:**

```
file1 file2 file3 file4 file5 file6 file7 file8 file9 file10
```

This confirms that both instances share the same EFS file system.

---

## 🔁 Step 10: Mount EFS Permanently

Edit the `/etc/fstab` file on **both instances**:

```bash
sudo vim /etc/fstab
```

Add the following line:

```bash
fs-0657508bc3c7fbf15:/ /root/data efs defaults 00
```

Save and verify:

```bash
sudo mount -av
```

---

## 📊 Architecture Diagram Creation

You can create the diagram using tools such as:

* **draw.io (diagrams.net)**
* **Lucidchart**
* **Microsoft Visio**
* **PowerPoint**

### Diagram Components

* Amazon EFS
* Two EC2 Instances in different AZs
* Security Group
* VPC
* NFS (Port 2049) connections

### Example Diagram Layout

```
                +----------------------+
                |      Amazon EFS      |
                +----------+-----------+
                           |
            +--------------+--------------+
            |                             |
+-----------+-----------+     +-----------+-----------+
|  EC2 Instance         |     |  EC2 Instance         |
|  NFS-server-01        |     |  NFS-server-02        |
|  AZ: us-south-1a      |     |  AZ: us-south-1b      |
+-----------------------+     +-----------------------+
```


---

## 📌 Key Benefits of Amazon EFS

* ✅ Highly available across multiple AZs
* ✅ Scalable and elastic storage
* ✅ Shared access for multiple EC2 instances
* ✅ Fully managed by AWS
* ✅ Supports NFSv4 with encryption in transit

---

## 📚 References

* [https://docs.aws.amazon.com/efs/](https://docs.aws.amazon.com/efs/)
* [https://docs.aws.amazon.com/ec2/](https://docs.aws.amazon.com/ec2/)

---

## 👨‍💻 Author

**Deep Bijwe**
Feel free to connect and contribute!

