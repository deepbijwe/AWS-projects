# 🐳 Portfolio Deployment on Docker — AWS EC2

> **Project by Deep Shyam Bijwe** | Cloud & IoT Engineer | Nagpur, India  
> Deployed at: `http://13.126.90.132:8080`

---

## 📐 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LOCAL MACHINE (Windows)                      │
│                                                                     │
│   index.html (Portfolio)  ──── scp ──────────────────────────────┐  │
│                                                                  │  │
│   my-mumbai.pem (Key Pair)  ── ssh  ──────────────────────────┐  │  │
└───────────────────────────────────────────────────────────────┼──┼──┘
                                                                │  │
                              Internet (Port 22 / Port 8080)    │  │
                                                                │  │ 
┌──────────────────────────────── AWS Cloud ────────────────────┼──┼──┐
│                                                               │  │  │
│  ┌──────────────────────────────────────────────────────────┐ │  │  │
│  │              EC2 Instance (t3.micro — ap-south-1a)       │ |  │  │
│  │              Public IP: 13.126.90.132                    │ │  │  │
│  │              OS: Ubuntu 24.04.4 LTS                      │ │  │  │
│  │                                                          │ │  │  │
│  │   SSH ◄────────────────────────────────────────────────── ◄┘  │  │
│  │                                                          │    │  │
│  │   /home/ubuntu/index.html  ◄──────── SCP ────────────── ◄─────   │
│  │          │                                             │         │
│  │          │  docker cp                                  │         │
│  │          ▼                                             │         │
│  │  ┌───────────────────────────────────────────────┐     │         │
│  │  │         Docker Engine (v29.1.3)               │     │         │
│  │  │                                               │     │         │
│  │  │  ┌─────────────────────────────────────────┐  │     │         │
│  │  │  │   Container: deepbijwe                  │  │     │         │
│  │  │  │   Image: nginx:latest                   │  │     │         │
│  │  │  │   Port: 8080 (host) → 80 (container)    │  │     │         │
│  │  │  │                                         │  │     │         │
│  │  │  │   /usr/share/nginx/html/index.html ◄──┐ │  │     │         │
│  │  │  └───────────────────────────────────────┼─┘  │     │         │
│  │  └──────────────────────────────────────────┼────┘     │         │
│  │                                             │          │         │
│  │   Security Group: Port 8080 open  ──────────┘          │         │
│  └────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
              ┌───────────────────────────┐
              │     Browser / Internet    │
              │  http://13.126.90.132:8080│
              │  (Portfolio Live!)        │
              └───────────────────────────┘
```

---

## 🚀 Step-by-Step Project Walkthrough

### Step 1 — Launch EC2 Instance on AWS

1. Go to **AWS Console → EC2 → Launch Instance**
2. Configure:
   - **Name:** `docker-test`
   - **AMI:** Ubuntu 24.04 LTS
   - **Instance Type:** `t3.micro` (Free Tier eligible)
   - **Key Pair:** Create/select `my-mumbai.pem`
   - **Region:** `ap-south-1` (Mumbai)
3. Under **Security Group**, open inbound ports:
   - `22` — SSH access
   - `8080` — Web traffic for Docker container
4. Click **Launch Instance**

> ✅ Instance running with Public IP: `13.126.90.132`

---

### Step 2 — SSH into the EC2 Instance

From your local Windows machine (Command Prompt / Terminal):

```bash
ssh -i my-mumbai.pem ubuntu@13.126.90.132
```

- Accept the host authenticity prompt by typing `yes`
- You're now inside the Ubuntu server

Switch to root:

```bash
sudo -i
```

---

### Step 3 — Install Docker on the Server

```bash
# Update packages
apt update -y

# Install Docker
apt install docker.io -y

# Verify installation
docker --version
# Output: Docker version 29.1.3, build 29.1.3-0ubuntu3~24.04.1
```

---

### Step 4 — Pull the Nginx Docker Image

```bash
docker pull nginx
```

Docker pulls the latest nginx image from Docker Hub:

```
Using default tag: latest
latest: Pulling from library/nginx
Status: Downloaded newer image for nginx:latest
```

Verify the image is available:

```bash
docker images
# IMAGE          ID            DISK USAGE
# nginx:latest   6e23479198b9  240MB
```

---

### Step 5 — Run the Docker Container

```bash
docker run -d --name deepbijwe -p 8080:80 nginx
```

| Flag | Meaning |
|------|---------|
| `-d` | Run in detached (background) mode |
| `--name deepbijwe` | Name the container |
| `-p 8080:80` | Map host port 8080 → container port 80 |
| `nginx` | Use the nginx image |

Verify container is running:

```bash
docker ps
# CONTAINER ID   IMAGE   PORTS                    NAMES
# db459e1d151e   nginx   0.0.0.0:8080->80/tcp     deepbijwe
```

---

### Step 6 — Transfer Portfolio HTML from Local to EC2

**Exit** the SSH session and run from your local machine:

```bash
# Exit the server first
exit

# Use SCP to transfer index.html to the EC2 instance
scp -i my-mumbai.pem index.html.html ubuntu@13.126.90.132:/home/ubuntu
```

> `scp` (Secure Copy Protocol) copies files over SSH.  
> The file lands at `/home/ubuntu/` on the server.

---

### Step 7 — Prepare the HTML File on the Server

SSH back into the instance:

```bash
ssh -i my-mumbai.pem ubuntu@13.126.90.132
```

Verify the file was transferred:

```bash
ls
# index.html.html
```

Rename it to the correct format:

```bash
cp index.html.html index.html
ls
# index.html   index.html.html
```

---

### Step 8 — Copy HTML into the Docker Container

Switch to root and copy the file into the running nginx container:

```bash
sudo -i

# Copy index.html into nginx's web root inside the container
docker cp /home/ubuntu/index.html deepbijwe:/usr/share/nginx/html

# Output: Successfully copied 239KB to deepbijwe:/usr/share/nginx/html
```

> Nginx serves files from `/usr/share/nginx/html/` inside the container.  
> The `docker cp` command injects the file without needing to rebuild the image.

---

### Step 9 — Access the Portfolio

Open a browser and navigate to:

```
http://13.126.90.132:8080
```

🎉 **Your portfolio is live!**

---

## 🔑 Key Commands Reference

| Command | Purpose |
|--------|---------|
| `ssh -i key.pem ubuntu@<IP>` | Connect to EC2 instance |
| `sudo -i` | Switch to root user |
| `docker pull nginx` | Download nginx image |
| `docker run -d --name <name> -p 8080:80 nginx` | Run container with port mapping |
| `docker ps` | List running containers |
| `docker exec -it <container> bash` | Open shell inside container |
| `scp -i key.pem file ubuntu@<IP>:/path` | Copy file from local to EC2 |
| `docker cp <src> <container>:<dest>` | Copy file into running container |
| `docker images` | List downloaded images |

---

## 🧠 Concepts Demonstrated

- **AWS EC2** — Provisioning a cloud Linux server
- **Security Groups** — Controlling inbound network access
- **SSH & SCP** — Secure remote access and file transfer
- **Docker** — Containerizing applications
- **Nginx** — Serving static web content
- **Port Mapping** — Exposing container services to the host
- **docker cp** — Injecting files into running containers without rebuilding

---

*Project completed on April 23, 2026 | Region: ap-south-1 (Mumbai)*
