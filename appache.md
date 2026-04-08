# 🚀 Hosting a Website on Apache2 Server (AWS Ubuntu)

## 🎯 Aim

To host a website on an Apache2 server using an AWS Ubuntu machine.

---

## 🧱 Architecture Diagram

```
                🌍 Internet
                     |
              [ Public IP ]
                     |
            -------------------
            |   AWS EC2       |
            |  Ubuntu Server  |
            |-----------------|
            |  Port 22 (SSH)  |
            |  Port 80 (HTTP) |
            -------------------
                     |
               Apache2 Server
                     |
             /var/www/html
                     |
               index.html
```

---

## ⚙️ Practical Steps

### 1️⃣ Launch Ubuntu Instance & Connect via SSH

* Launch an Ubuntu EC2 instance on AWS

* Allow:

  * Port **22** (SSH)
  * Port **80** (HTTP)

* Connect using SSH from local terminal:

```bash
ssh -i key.pem ubuntu@<public-ip>
```

---

### 2️⃣ Update Server

```bash
apt update -y
```

---

### 3️⃣ Install Apache2

```bash
apt install apache2 -y
```

---

### 4️⃣ Check Apache Service Status

```bash
systemctl status apache2
```

---

### 5️⃣ Start Service (if not running)

```bash
systemctl start apache2
```

* Check status again:

```bash
systemctl status apache2
```

* Copy the **Public IP** and open in browser to verify Apache default page

---

### 6️⃣ Host Website (index.html)

```bash
cd /var/www/html
ls
cat > index.html
```

* Paste your HTML code:

```html
<h1>Welcome to My Website</h1>
<p>This is hosted on Apache2 server.</p>
```

* Save file and exit

* Open browser and hit:

```
http://<public-ip>
```

---

## ✅ Conclusion

You successfully hosted your website on an Apache server on AWS using an Ubuntu machine.

---
