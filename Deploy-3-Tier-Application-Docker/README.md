# 🐳 3-Tier Application Deployment on Docker
### CLOUDBLITZ Student Registration App — AWS EC2 + Docker

> Deploy a full-stack **React Frontend → Spring Boot Backend → MariaDB Database** using Docker containers on AWS EC2. By the end, the CLOUDBLITZ Student Registration portal will be live and accessible from a browser.

---

## 🏗️ Architecture Diagram

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                    AWS EC2 — docker-3-tier                               │
  │              t3.micro | Ubuntu 24.04 LTS | us-east-1 (N. Virginia)       │
  │              Public IP: 54.198.31.71                                     │
  │                                                                          │
  │  ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐      │
  │  │   TIER 1         │   │   TIER 2         │   │   TIER 3         │      │
  │  │   FRONTEND       │──►│   BACKEND        │──►│   DATABASE       │      │
  │  │                  │   │                  │   │                  │      │
  │  │  node:24-alpine  │   │ maven:3.8.3-     │   │ mariadb:latest   │      │
  │  │  + apache2       │   │ openjdk-17       │   │                  │      │
  │  │                  │   │ Spring Boot JAR  │   │ student_db       │      │
  │  │  frontend-cont   │   │ backend-cont     │   │ database         │      │
  │  │  Port: 80        │   │ Port: 8080       │   │ Port: 3306       │      │
  │  └──────────────────┘   └──────────────────┘   └──────────────────┘      │
  │           │                      │                      │                │
  │           └──────────────────────┴──────────────────────┘                │
  │                         Docker Bridge Network                            │
  │                      DB Container IP: 172.17.0.2                         │
  │                                                                          │
  │  Security Group:  :22 SSH │ :80 HTTP │ :8080 Spring Boot │ :3306 DB      │
  └──────────────────────────────────────────────────────────────────────────┘
                                      │
                               ┌──────▼──────┐
                               │   Browser   │
                               │ 54.198.31.71│
                               └─────────────┘
```

---

## 📋 Instance Details (from AWS Console)

| Field              | Value                                        |
|--------------------|----------------------------------------------|
| Instance Name      | `docker-3-tier`                              |
| Instance ID        | `i-06fd44fca32d2caca`                        |
| Instance Type      | `t3.micro`                                   |
| AMI                | Ubuntu 24.04 LTS                             |
| Region             | US East — N. Virginia (`us-east-1`)          |
| Public IPv4        | `54.198.31.71`                               |
| Private IPv4       | `172.31.47.231`                              |
| Key Pair           | `n.virginia.pem`                             |
| VPC                | `vpc-03d48d297c0354076` (Default)            |
| State              | ✅ Running                                    |

---

## 🔒 Security Group — Inbound Rules

| Type     | Protocol | Port  | Source     | Purpose                  |
|----------|----------|-------|------------|--------------------------|
| SSH      | TCP      | 22    | My IP      | Remote terminal access   |
| HTTP     | TCP      | 80    | 0.0.0.0/0  | Frontend (Apache2)       |
| Custom   | TCP      | 8080  | 0.0.0.0/0  | Backend (Spring Boot)    |
| Custom   | TCP      | 3306  | 0.0.0.0/0  | MariaDB Database         |

---

## 🚀 Step 1 — Launch EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instance**
2. Set **Name**: `3-Tier-docker`
3. Select **AMI**: Ubuntu 24.04 LTS
4. Select **Instance Type**: `t3.micro`
5. Choose **Key Pair**: `n.virginia.pem`
6. Set **VPC**: Default; configure security group inbound rules as shown above
7. Click **Launch Instance** ✅

> 📸 **Screenshot 1:** EC2 console confirming the `docker-3-tier` instance is in **Running** state with Public IP `54.198.31.71`, Instance ID `i-06fd44fca32d2caca`, and type `t3.micro` in N. Virginia.

---

## 🔧 Step 2 — SSH In & Install Docker

### Connect to the Instance
```bash
ssh -i n.virginia.pem ubuntu@54.198.31.71
```

### Install Docker
```bash
apt update && apt install docker.io -y
docker --version
# Docker version 29.1.3-0ubuntu4.1
```

> 📸 **Screenshot 2:** Terminal showing `Setting up docker.io (29.1.3-0ubuntu4.1)` with systemd symlinks created for `docker.service` and `docker.socket` — Docker installed successfully on the EC2 instance.

### Run the MariaDB Container
```bash
docker run -d -it --name database -p 3306:3306 mariadb
```

### Verify Container Status
```bash
docker ps -a
# CONTAINER ID  IMAGE    STATUS           PORTS                   NAMES
# 975f6f4503ca  mariadb  Up 19 seconds    0.0.0.0:3306->3306/tcp  database
```

### Access the MariaDB Shell Inside the Container
```bash
docker exec -it database mariadb -pdeep123
```

> ⚠️ **Gotcha:** Running without the password flag gives `ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)`. Always use `-pdeep123` (no space between `-p` and the password).

### Create the Application Database
```sql
MariaDB [(none)]> create database student_db;
-- Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> show databases;
-- +--------------------+
-- | Database           |
-- +--------------------+
-- | information_schema |
-- | mysql              |
-- | performance_schema |
-- | student_db         |  ✅
-- | sys                |
-- +--------------------+
-- 5 rows in set (0.002 sec)
```

> 📸 **Screenshot 3:** MariaDB shell (version `12.2.2-MariaDB-ubu2404`) showing `student_db` successfully created, confirmed by `show databases` listing all 5 databases including `student_db`.

---

## ⚙️ Step 3 — Clone Repository & Configure Backend

### Clone the EASY_CRUD Repository
```bash
git clone https://github.com/abhilashmwaghmare/EASY_CRUD.git
ls
# EASY_CRUD  snap

cd EASY_CRUD/backend
ls
# Dockerfile  Readme.md  application.properties  backend-yaml  mvnw  mvnw.cmd  pom.xml  src
```

> 📸 **Screenshot 4:** Git clone output — 272 objects received, 1.04 MiB downloaded. The `ls` inside `EASY_CRUD/backend` shows `Dockerfile`, `application.properties`, `pom.xml`, `src/`.

### Get the MariaDB Container's Internal IP
```bash
docker ps -a
# Container ID starts with: 975f6f4503ca

docker inspect 975f6f4503ca | grep "IP"
# "IPAddress": "172.17.0.2"   ← Copy this
```

> 📸 **Screenshot 4 (cont.):** `docker inspect` output showing `"IPAddress": "172.17.0.2"` — the Docker bridge network IP used for backend-to-database communication inside the host.

### Configure `application.properties`
```bash
vim application.properties
```

Paste the following configuration:

```properties
server.port=8080
spring.datasource.url=jdbc:mariadb://172.17.0.2:3306/student_db
spring.datasource.username=root
spring.datasource.password=deep123
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

Save and exit vim:
```
Esc → :wq → Enter
```

> 📸 **Screenshot 5:** Vim editor showing the complete `application.properties` — JDBC URL set to `172.17.0.2:3306/student_db`, username `root`, password `deep123`, with JPA auto DDL and SQL logging enabled.

---

## 🐋 Step 4 — Review Backend Dockerfile & Build Image

### Review the Dockerfile
```bash
vim Dockerfile
```

**Backend Dockerfile (actual content from repo):**
```dockerfile
FROM maven:3.8.3-openjdk-17

COPY . /opt/
WORKDIR /opt

# Replace the bundled config with our custom application.properties
RUN rm -rf src/main/resources/application.properties
RUN cp -f application.properties src/main/resources/application.properties

# Build the Spring Boot JAR (skip tests for speed)
RUN mvn clean package -DskipTests

WORKDIR /opt/target/

EXPOSE 8080

CMD ["java", "-jar", "student-registration-backend-0.0.1-SNAPSHOT.jar"]
```

> 📸 **Screenshot 6:** Vim editor displaying the backend `Dockerfile` — 25 lines, 532B. Uses `maven:3.8.3-openjdk-17` as the builder base, injects the custom `application.properties`, runs `mvn clean package`, and starts the app JAR on port 8080.

### Build the Backend Docker Image
```bash
docker build -t backend:latest .
```

> 📸 **Screenshot 7:** Docker build progress — pulling `maven:3.8.3-openjdk-17` (sha256: `8a66581a...`), executing steps: `COPY . /opt/` → `WORKDIR /opt` → removing old config → copying new `application.properties` → running Maven package.

---

## ✅ Step 5 — Verify Backend Image

```bash
docker images
```

Expected output:
```
IMAGE                  ID              DISK USAGE    CONTENT SIZE
backend:latest         9b4120891150    1.52 GB       550 MB
mariadb:latest         e0236fc6386e    467 MB        111 MB    [In Use]
maven:3.8.3-openjdk-17 8a66581a0777    1.24 GB       422 MB
```

> 📸 **Screenshot 8:** `docker images` showing all three images: `backend:latest` (1.52 GB), `mariadb:latest` (467 MB, marked **In Use** by the running container), and `maven:3.8.3-openjdk-17` (1.24 GB build image).

---

## 🚦 Step 6 — Run the Backend Container

```bash
docker run -d -it --name backend-cont -p 8080:8080 backend:latest
```

### Verify Both Containers Running
```bash
docker ps -a
```

```
CONTAINER ID  IMAGE           STATUS        PORTS                   NAMES
ec4c51215ae8  backend:latest  Up 3 seconds  0.0.0.0:8080->8080/tcp  backend-cont
975f6f4503ca  mariadb         Up 48 min     0.0.0.0:3306->3306/tcp  database
```

> 📸 **Screenshot 8 (cont.):** `docker ps -a` showing `backend-cont` (Up 3 seconds, port 8080) and `database` (Up 48 minutes, port 3306) — Tier 2 and Tier 3 are both live.

---

## 🌐 Step 7 — Configure & Build the Frontend

### Navigate to the Frontend Directory
```bash
cd EASY_CRUD/frontend
ls
# Dockerfile  README.md  eslint.config.js  frontend-yaml  index.html  package.json  public  src  vite.config.js

ls -a
# Shows hidden files including: .env  .dockerignore  .gitignore
```

### Update the `.env` File with Your EC2 Public IP
```bash
vim .env
```

```env
VITE_API_URL = "http://54.198.31.71:8080/api"
```

Save and exit:
```
Esc → :wq → Enter
```

> 📸 **Screenshot 13:** Vim editor showing `.env` file with `VITE_API_URL = "http://<EC2-IP>:8080/api"` — this tells the React/Vite frontend where to reach the Spring Boot backend API.

---

## 🏗️ Step 8 — Build & Run the Frontend Container

### Review the Frontend Dockerfile
```bash
vim Dockerfile
```

**Frontend Dockerfile (actual content):**
```dockerfile
FROM node:24-alpine

COPY . /opt/
WORKDIR /opt

# Install dependencies and build Vite/React app
RUN npm install && npm run build

# Install Apache2 web server
RUN apk update && apk add apache2

# Deploy Vite build output to Apache web root
RUN rm -f /var/www/localhost/htdocs/*
RUN cp -rf dist/* /var/www/localhost/htdocs/

EXPOSE 80

CMD ["httpd", "-D", "FOREGROUND"]
```

> 📸 **Screenshot 9:** Vim editor showing the frontend `Dockerfile` — 21 lines, 335B. Uses `node:24-alpine` for building the Vite app, then installs Apache2 via `apk`, serves the static `dist/` output on port 80.

### Build the Frontend Image
```bash
docker build -t frontend:latest .
```

> 📸 **Screenshot 10:** Build starts by pulling `node:24-alpine` (sha256: `d1b3b4da11ee...`), executing 9 steps: `COPY` → `WORKDIR` → `npm install && npm run build` → `apk add apache2`.

> 📸 **Screenshot 11:** Build completion — Vite bundles assets (`bgimg.png` 783KB, `index.css` 2.6KB, `index.js` 199KB), Apache2 installed (6 packages, 14.6 MiB), `dist/` copied to `htdocs/`, and final output: `Successfully tagged frontend:latest` ✅

### Run the Frontend Container
```bash
docker run -dit --name frontend-cont -p 80:80 frontend:latest
```

### Final Verification — All 3 Containers Up
```bash
docker ps -a
```

```
CONTAINER ID  IMAGE             STATUS    PORTS                   NAMES
xxxxxxxxxxxx  frontend:latest   Up        0.0.0.0:80->80/tcp      frontend-cont  ✅
ec4c51215ae8  backend:latest    Up        0.0.0.0:8080->8080/tcp  backend-cont   ✅
975f6f4503ca  mariadb           Up        0.0.0.0:3306->3306/tcp  database       ✅
```

---

## 🎉 Access the Application

Open your browser and visit:
```
http://54.198.31.71
```

> 📸 **Screenshot 12:** Browser at `http://54.198.31.71` showing the **CLOUDBLITZ Student Registration** portal — black-and-orange themed UI (Greamio Technologies × CLOUDBLITZ branding). The registration form accepts Name, Email, Course, Highest Education, Percentage, Branch, and Mobile Number. A registered student **Deep Bijwe** (Course: CDEC, Class: BE, Branch: ENTC, Percentage: 73, Mobile: 07219374775) appears in the table below with a Delete action — confirming the complete 3-tier data flow is working. ✅

---

## 📊 Final Summary

### Containers

| Tier   | Container Name | Image                  | Port | Technology              |
|--------|---------------|------------------------|------|-------------------------|
| Tier 1 | `frontend-cont`| `frontend:latest`      | 80   | React + Vite + Apache2  |
| Tier 2 | `backend-cont` | `backend:latest`       | 8080 | Spring Boot (Java 17)   |
| Tier 3 | `database`     | `mariadb:latest`       | 3306 | MariaDB 12.2.2          |

### Docker Images

| Image                   | Size    | Role               |
|-------------------------|---------|--------------------|
| `backend:latest`        | 1.52 GB | Spring Boot app    |
| `maven:3.8.3-openjdk-17`| 1.24 GB | Backend build base |
| `mariadb:latest`        | 467 MB  | Database server    |
| `frontend:latest`       | ~200 MB | React + Apache     |
| `node:24-alpine`        | ~70 MB  | Frontend build base|

---

## 🔍 Docker Quick Reference

| Command                                         | Description                          |
|-------------------------------------------------|--------------------------------------|
| `docker ps -a`                                  | List all containers (running/stopped)|
| `docker images`                                 | List all local images                |
| `docker logs <container>`                       | View container output logs           |
| `docker exec -it <container> bash`              | Open shell inside container          |
| `docker inspect <id> \| grep "IP"`              | Get container's internal IP          |
| `docker stop <container>`                       | Stop a running container             |
| `docker rm <container>`                         | Remove a stopped container           |
| `docker rmi <image>`                            | Delete a local image                 |
| `docker build -t <name>:<tag> .`                | Build image from Dockerfile          |
| `docker run -dit --name <n> -p host:cont image` | Run container detached               |

---

## ⚠️ Common Gotchas & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| `Access denied for user 'root'` on DB login | Missing password flag | Use `mariadb -pdeep123` (no space after `-p`) |
| `unknown shorthand flag: 'a' in -a` | Typo: `docker pa -a` | Correct: `docker ps -a` |
| `unknown shorthand flag: 't' in -t` | Typo: `docker built` | Correct: `docker build` |
| Frontend shows blank page | Wrong IP in `.env` | Set `VITE_API_URL` to the EC2 **public** IP, not private |
| Backend can't connect to DB | Container IP changed | Re-run `docker inspect` to get the current IP |

---

> 💡 **Production Tip:** Hardcoding the database container IP (`172.17.0.2`) breaks when containers restart, as Docker may assign a different IP. Use a named Docker network so containers can reference each other by name:
>
> ```bash
> docker network create app-network
> docker run --network app-network --name database -p 3306:3306 mariadb
> docker run --network app-network --name backend-cont -p 8080:8080 backend:latest
> # In application.properties:
> # spring.datasource.url=jdbc:mariadb://database:3306/student_db
> ```