# 📸 Immich Self-Hosted Photo Server — Docker Setup Guide

> **Immich** হলো একটি self-hosted photo & video backup solution, যা Google Photos-এর মতো কাজ করে। নিজের server-এ সম্পূর্ণ privacy সহ ছবি ও ভিডিও backup রাখা যায়।

---

## 📋 Table of Contents

- [Requirements](#-requirements)
- [Project Structure](#-project-structure)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Running the Server](#-running-the-server)
- [Accessing Immich](#-accessing-immich)
- [First Time Setup](#-first-time-setup)
- [Useful Commands](#-useful-commands)
- [Updating Immich](#-updating-immich)
- [Troubleshooting](#-troubleshooting)
- [Security Tips](#-security-tips)

---

## ✅ Requirements

শুরু করার আগে নিচের জিনিসগুলো তোমার system-এ installed থাকতে হবে:

| Tool           | Minimum Version              | Check Command            |
| -------------- | ---------------------------- | ------------------------ |
| Docker         | 24.0+                        | `docker --version`       |
| Docker Compose | 2.0+                         | `docker compose version` |
| RAM            | 4 GB+                        | —                        |
| Storage        | যত ছবি রাখতে চাও তার দ্বিগুণ | —                        |

### Docker Install করো (যদি না থাকে)

**Ubuntu/Debian:**

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

**Windows:**
[Docker Desktop](https://www.docker.com/products/docker-desktop/) download করে install করো।

**macOS:**
[Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/) download করে install করো।

---

## 📁 Project Structure

একটি নতুন folder তৈরি করো এবং নিচের structure follow করো:

```
immich/
├── docker-compose.yml    ← main config file
├── config/               ← auto-created by Docker
├── photos/               ← তোমার photos এখানে store হবে
└── postgres/             ← database data (auto-created)
```

---

## 🚀 Installation

### Step 1 — Project Folder তৈরি করো

```bash
mkdir immich-server
cd immich-server
```

### Step 2 — `docker-compose.yml` File তৈরি করো

```bash
nano docker-compose.yml
```

নিচের পুরো code টি paste করো:

```yaml
version: "3.8"
services:
  immich:
    image: ghcr.io/imagegenius/immich:latest
    container_name: immich
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Dhaka
      - DB_HOSTNAME=postgres14
      - DB_USERNAME=postgres
      - DB_PASSWORD=postgres
      - DB_DATABASE_NAME=immich
      - REDIS_HOSTNAME=valkey
      - DB_PORT=5432
      - REDIS_PORT=6379
      - SERVER_HOST=0.0.0.0
      - SERVER_PORT=8080
    volumes:
      - ./config:/config
      - ./photos:/photos
    ports:
      - 8080:8080
    depends_on:
      - valkey
      - postgres14
    restart: unless-stopped

  valkey:
    image: valkey/valkey:8-bookworm
    container_name: valkey
    ports:
      - 6379:6379
    restart: unless-stopped

  postgres14:
    image: ghcr.io/immich-app/postgres:14-vectorchord0.3.0-pgvectors0.2.0
    container_name: postgres14
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: immich
    volumes:
      - ./postgres:/var/lib/postgresql/data
    ports:
      - 5432:5432
    restart: unless-stopped
```

`Ctrl+O` দিয়ে save করো, `Ctrl+X` দিয়ে বের হও।

---

## ⚙️ Configuration

### Environment Variables — কী কী পরিবর্তন করবে

| Variable           | Default Value | বিবরণ                                                       |
| ------------------ | ------------- | ----------------------------------------------------------- |
| `PUID`             | `1000`        | Linux user ID (যদি permission issue হয় `id -u` দিয়ে দেখো) |
| `PGID`             | `1000`        | Linux group ID (`id -g` দিয়ে দেখো)                         |
| `TZ`               | `Asia/Dhaka`  | Timezone                                                    |
| `DB_PASSWORD`      | `postgres`    | ⚠️ Production-এ অবশ্যই পরিবর্তন করো!                        |
| `DB_USERNAME`      | `postgres`    | Database username                                           |
| `DB_DATABASE_NAME` | `immich`      | Database name                                               |
| `SERVER_PORT`      | `8080`        | Immich web port                                             |

> **⚠️ Important:** Production server-এ `DB_PASSWORD` এবং `POSTGRES_PASSWORD` অবশ্যই strong password দিয়ে পরিবর্তন করো।

### PUID এবং PGID বের করার উপায়

```bash
id -u   # PUID বের করবে
id -g   # PGID বের করবে
```

---

## ▶️ Running the Server

### Step 1 — Photos Folder তৈরি করো

```bash
mkdir -p photos config
```

### Step 2 — Docker Containers Start করো

```bash
docker compose up -d
```

প্রথমবার চালালে Docker images download হবে (কিছুটা সময় লাগবে, internet speed এর উপর নির্ভর করে)।

### Step 3 — Container Status চেক করো

```bash
docker compose ps
```

সব container এর status `Up` থাকলে সফলভাবে চালু হয়েছে।

**Expected Output:**

```
NAME          IMAGE                          STATUS
immich        ghcr.io/imagegenius/immich     Up
postgres14    ghcr.io/immich-app/postgres    Up
valkey        valkey/valkey:8-bookworm       Up
```

---

## 🌐 Accessing Immich

Server চালু হওয়ার পর browser-এ যাও:

| Access Type            | URL                       |
| ---------------------- | ------------------------- |
| **Local Machine**      | `http://localhost:8080`   |
| **Same Network (LAN)** | `http://<তোমার-IP>:8080`  |
| **Remote Server**      | `http://<server-IP>:8080` |

তোমার local IP বের করার উপায়:

```bash
# Linux/macOS
hostname -I

# Windows
ipconfig
```

---

## 🆕 First Time Setup

### 1. Admin Account তৈরি করো

প্রথমবার `http://localhost:8080` open করলে একটি registration page দেখাবে।

- **Name:** তোমার নাম
- **Email:** তোমার email (এটাই login ID হবে)
- **Password:** strong password দাও

### 2. Mobile App Connect করো

Immich-এর official mobile app download করো:

- **Android:** [Google Play Store](https://play.google.com/store/apps/details?id=app.alextran.immich)
- **iOS:** [App Store](https://apps.apple.com/app/immich/id1613945652)

App এ server URL দাও: `http://<তোমার-IP>:8080`

### 3. Library Setup করো

Web UI তে গিয়ে:

1. **Settings** → **Libraries** → **External Library** তৈরি করো
2. Path: `/photos` (এটা Docker volume এর সাথে linked)

---

## 🛠️ Useful Commands

### Logs দেখো

```bash
# সব container এর logs
docker compose logs -f

# শুধু Immich এর logs
docker compose logs -f immich

# শুধু Database এর logs
docker compose logs -f postgres14
```

### Container Restart করো

```bash
# সব services restart
docker compose restart

# শুধু একটি service restart
docker compose restart immich
```

### Container বন্ধ করো

```bash
# বন্ধ করো কিন্তু data রেখে দাও
docker compose down

# সব কিছু সহ বন্ধ করো (data থাকবে, volumes মুছবে না)
docker compose down --remove-orphans
```

### Container চালু করো

```bash
docker compose up -d
```

### Resource Usage দেখো

```bash
docker stats
```

---

## 🔄 Updating Immich

নতুন version আসলে update করার উপায়:

```bash
# নতুন images pull করো
docker compose pull

# Containers restart করো নতুন image দিয়ে
docker compose up -d

# পুরনো images মুছে দাও (optional, space বাঁচাতে)
docker image prune -f
```

---

## 🔧 Troubleshooting

### ❌ Port Already in Use Error

```bash
# কোন process port 8080 use করছে দেখো
sudo lsof -i :8080
# অথবা
sudo netstat -tulpn | grep 8080
```

সমাধান: `docker-compose.yml` এ port পরিবর্তন করো, যেমন `9090:8080`

---

### ❌ Database Connection Error

```bash
# Postgres container এর logs দেখো
docker compose logs postgres14

# Container এর ভেতরে ঢুকে check করো
docker exec -it postgres14 psql -U postgres -d immich
```

সমাধান: সব container বন্ধ করে আবার চালু করো:

```bash
docker compose down
docker compose up -d
```

---

### ❌ Permission Denied Error (Photos Folder)

```bash
# Photos folder এর permission ঠিক করো
sudo chown -R 1000:1000 ./photos ./config
sudo chmod -R 755 ./photos ./config
```

---

### ❌ Container বারবার Restart হচ্ছে

```bash
# Logs দেখো কারণ বুঝতে
docker compose logs --tail=50 immich
```

---

### ❌ Immich খুব Slow লাগছে

প্রথমবার চালানোর পর Immich সব ছবির thumbnail generate করে, এই সময় slow হওয়া স্বাভাবিক। কিছুক্ষণ অপেক্ষা করো।

---

## 🔒 Security Tips

Production server বা internet-এ expose করতে চাইলে অবশ্যই:

1. **Password পরিবর্তন করো**

   ```yaml
   DB_PASSWORD: "তোমার_strong_password_এখানে"
   POSTGRES_PASSWORD: "তোমার_strong_password_এখানে"
   ```

2. **Firewall Setup করো**

   ```bash
   # শুধু দরকারি ports খোলো
   sudo ufw allow 8080/tcp
   sudo ufw enable
   ```

3. **Reverse Proxy ব্যবহার করো** (Nginx বা Caddy দিয়ে HTTPS enable করো)

4. **Regular Backup নাও**

   ```bash
   # Photos backup
   cp -r ./photos /backup/photos-$(date +%Y%m%d)

   # Database backup
   docker exec postgres14 pg_dump -U postgres immich > backup-$(date +%Y%m%d).sql
   ```

---

## 📞 Support & Resources

| Resource                 | Link                                         |
| ------------------------ | -------------------------------------------- |
| Immich Official Docs     | https://immich.app/docs                      |
| GitHub Repository        | https://github.com/immich-app/immich         |
| Community Discord        | https://discord.immich.app                   |
| ImageGenius Docker Image | https://github.com/imagegenius/docker-immich |

---

> **💡 Tip:** Immich নিয়মিত update হয়। GitHub-এ star দিয়ে রাখো নতুন features সম্পর্কে জানতে।

---

_Made with ❤️ for self-hosters_
