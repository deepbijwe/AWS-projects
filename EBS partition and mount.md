# 🚀 Attach EBS to EC2, Create Partition & Mount Directory (AWS Ubuntu)

## 🎯 Aim

Attach an EBS volume to an EC2 server, create partitions, and mount it to a directory.

---

## 🧱 Architecture Diagram

```
                ☁️ AWS Cloud
                     |
             -------------------
             |     EC2         |
             | Ubuntu Server   |
             -------------------
                     |
         -------------------------
         |   EBS Volume (100 GiB) |
         -------------------------
                     |
              Partition (20 GiB)
                     |
               /dev/nvme1n1p1
                     |
                 Mounted to
                   /backup
```

---

## ⚙️ Practical Steps

### 1️⃣ Create & Attach EBS Volume

* Create **100 GiB EBS Volume**
* Attach it to existing EC2 instance

---

### 2️⃣ Verify Volume Attachment

```bash
lsblk
```

* Shows attached volumes

---

### 3️⃣ Create Partition

```bash
fdisk /dev/nvme1n1
```

Inside fdisk:

```
n  → new partition  
p  → primary  
enter  
enter  
+20G  
```

* Creates **20GB partition**
* Verify:

```bash
lsblk
```

---

### 4️⃣ Create Filesystem

```bash
mkfs.ext4 /dev/nvme1n1p1
```

---

### 5️⃣ Create Mount Directory

```bash
mkdir /backup
```

---

### 6️⃣ Mount Partition

```bash
mount /dev/nvme1n1p1 /backup
```

---

### 7️⃣ Verify Mount

```bash
df -hT
```

---

### 8️⃣ Permanent Mount using fstab

#### Get UUID

```bash
blkid
```

#### Edit fstab

```bash
vim /etc/fstab
```

Add:

```
UUID="        "  ext4  /root/backup  defaults  0  0
```

Save & Exit:

```
:wq!
```

---

### 🔍 Verification

```bash
umount /dev/nvme1n1p1
mount -a
```

* Mounts all entries from fstab

---

## ✅ Conclusion

You successfully:

* Attached EBS to EC2
* Created partition
* Formatted disk
* Mounted it to directory
* Configured permanent mount using fstab

---
