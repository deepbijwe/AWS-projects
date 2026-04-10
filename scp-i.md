
# 🔐 Securely Copy Files to AWS EC2 Using "`scp-i`"

## 🎯 Aim
To securely copy files from a local machine to an AWS EC2 instance using the Linux `scp -i` command.

---

## 🛠️ Prerequisites

- An AWS account.
- An EC2 instance running **Ubuntu**.
- Port **22 (SSH)** enabled in the instance's Security Group.
- A **.pem key pair** for authentication.
- `ssh` and `scp` installed on your local machine.

---

## 📌 Step 1: Create an EC2 Instance

1. Log in to the **AWS Management Console**.
2. Launch a new EC2 instance.
3. Choose the **Ubuntu AMI**.
4. Create or select an existing **key pair (.pem)**.
5. Ensure that **port 22 (SSH)** is allowed in the security group.
6. Note the **public IP address** of the instance.

---

## 📌 Step 2: Generate an SSH Key Pair (Optional)

If you don’t already have an SSH key pair, create one using:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
````

This command generates:

* `Linux` (private key)
* `Linux.pub` (public key)

If you already have a key pair, you can skip this step.

---

## 📌 Step 3: Use `scp -i` to Copy the Public Key to the Server

### 🔹 Command Syntax

```bash
scp -i <pem-file> <source-file> <username>@<public-ip>:<destination-path>
```

### 🔹 Parameter Explanation

| Parameter          | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `.pem`             | Key pair used to authenticate with the EC2 instance          |
| `source-file`      | The file you want to copy (e.g., `Linux.pub`)                |
| `username`         | Default username for the AMI (e.g., `ubuntu` for Ubuntu)     |
| `public-ip`        | Public IP address of the EC2 instance                        |
| `:`                | Separator between the remote host and destination path       |
| `destination-path` | Directory on the remote server where the file will be stored |

### 🔹 Example Command

```bash
scp -i mymumbai-key.pem Linux.pub ubuntu@18.61.44.169:/home/ubuntu
```

### 🔹 Confirmation

* When connecting for the first time, you may see a prompt:

```text
The authenticity of host '18.61.44.169' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

* Type `yes` and press **Enter**.
* The file will be securely copied to the EC2 instance.

---

## 📌 Step 4: Verify the File on the Server

Connect to the EC2 instance:

```bash
ssh -i mymumbai-key.pem ubuntu@18.61.44.169
```

Check if the file exists:

```bash
ls -l /home/ubuntu
```

---

## 📊 Command Flow Diagram
+------------------+         SCP (Secure Copy)        +----------------------+
|  Local Machine   |  ----------------------------->  |  AWS EC2 Instance    |
|                  |     Using .pem authentication    |                      |
|  Linux.pub File  |                                  |  /home/ubuntu        |
+------------------+                                  +----------------------+

## 🖼️ Suggested Images for the Repository

Add the following screenshots to an `images/` directory in your repository:

1. **EC2 Instance Creation**

   * Filename: `images/ec2-instance-creation.png`
   * Description: Shows the selection of the Ubuntu AMI and key pair.

2. **Security Group Configuration**

   * Filename: `images/security-group-ssh.png`
   * Description: Demonstrates port 22 (SSH) being allowed.

3. **Public IP Address**

   * Filename: `images/ec2-public-ip.png`
   * Description: Displays where to find the public IP of the instance.

4. **SCP Command Execution**

   * Filename: `images/scp-command.png`
   * Description: Terminal output showing the successful execution of the `scp` command.

5. **File Verification on EC2**

   * Filename: `images/file-verification.png`
   * Description: `ls -l` output confirming the file transfer.

### 📌 Example of Embedding Images in Markdown

```markdown
## 📷 EC2 Instance Creation
![EC2 Instance Creation](images/ec2-instance-creation.png)

## 📷 SCP Command Execution
![SCP Command](images/scp-command.png)
```
---

## 🔒 Security Notes

* Never share your `.pem` private key publicly.
* Set appropriate permissions for the key file:

```bash
chmod 400 mymumbai-key.pem
```

* Ensure that only trusted IPs are allowed in the security group.

---

## 📝 Conclusion

Using the `scp -i` command, you can securely transfer files from your local machine to an AWS EC2 instance. This method leverages SSH encryption and key-based authentication, ensuring safe and efficient file transfers. It is a reliable approach for managing files on remote Linux servers such as Ubuntu.

---

## 📚 References

* AWS EC2 Documentation: [https://docs.aws.amazon.com/ec2/](https://docs.aws.amazon.com/ec2/)
* SCP Manual: [https://man.openbsd.org/scp](https://man.openbsd.org/scp)

