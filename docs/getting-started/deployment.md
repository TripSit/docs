# Host Deployment Guide

This guide walks an Ubuntu Linux administrator step-by-step through configuring a host machine and deploying the TripSit server stack.

---

## 📋 System Requirements

- **Operating System**: Ubuntu 22.04 LTS or 24.04 LTS (64-bit).
- **CPU**: 4+ cores recommended.
- **RAM**: 16 GB+ recommended (given Matrix, Sonar, Moodle, and PostgreSQL instances).
- **Disk**: 100 GB+ SSD/NVMe storage mounted at `/` or `/docker`.
- **Public IP**: Static IPv4 address (and optionally IPv6).

---

## 🛠️ Step 1: Ubuntu Host Preparation

### 1. Update Package Index and Install Dependencies
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git ufw jq ca-certificates gnupg lsbrelease
```

### 2. Install Docker Engine & Docker Compose Plugin
Install the official Docker repository on Ubuntu:
```bash
# Add Docker's official GPG key:
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verify installation:
```bash
docker --version
docker compose version
```

---

## 📁 Step 2: Create Host Directory Structure

As described in the [Architecture Overview](overview.md), TripSit relies on specific paths under `/docker`.

Execute as root or with `sudo`:
```bash
sudo mkdir -p /docker/appdata
sudo mkdir -p /docker/custom
sudo mkdir -p /docker/logs
sudo mkdir -p /docker/scripts
sudo mkdir -p /docker/secrets
sudo mkdir -p /docker/shared

# Lock down secrets folder permissions
sudo chmod 700 /docker/secrets
```

---

## 🔑 Step 3: Global Environment Setup (`.env`)

Each compose stack references global system and environment variables stored in a central `.env` file located in the root repository folder (`/home/moonbear/UAT/.env` or working directory).

!!! note "Security Rule"
    Never commit `.env` files to git repositories. Always use sample placeholders when creating configuration templates.

### Example Environment Configuration Template (`.env`)
Create a `.env` file in your repository root with the following parameters:

```env
# System Parameters
TZ=UTC
PUID=1000
PGID=1000
DOCKERDIR=/docker

# Domain Setup
DNS_DOMAIN=tripsit.me
BOT_DOMAIN=tripsit.me

# Authentication / OAuth
GOOGLE_CLIENT_ID=<your-google-client-id>.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=<your-google-client-secret>
OAUTH_SECRET=<random-32-byte-secret>
OAUTH_WHITELIST=user1@example.com,user2@example.com

# Cloudflare Integration
CLOUDFLARE_EMAIL=admin@tripsit.me
CLOUDFLARE_API_KEY=<your-cloudflare-api-key>

# Database Passwords
CROWDSEC_DB_PASS=<secure-password>
TRIPBOT_DB_PASS=<secure-password>
POSTGRES_PASSWORD=<secure-password>
MARIADB_ROOT_PASSWORD=<secure-password>
```

---

## 🚀 Step 4: Launching Docker Compose Stacks

TripSit services are launched modularly using standard `docker compose` commands specifying the target `.yml` file.

### 1. Launch Core Infrastructure (Crucial First Step!)
Core infrastructure (Traefik, Socket Proxy, OAuth) must be running before dependent stacks start:

```bash
# Navigate to repository directory
cd /home/moonbear/UAT

# Validate compose syntax
docker compose -f docker-compose.core.yml config

# Start core services in detached mode
docker compose -f docker-compose.core.yml up -d
```

Check running containers:
```bash
docker compose -f docker-compose.core.yml ps
```

### 2. Launch Administrative Applications
```bash
docker compose -f docker-compose.apps.yml up -d
```

### 3. Launch Communication & Chat Services
```bash
docker compose -f docker-compose.irc.yml up -d
docker compose -f docker-compose.matrix.yml up -d
```

### 4. Launch Web, Bots & Knowledge Bases
```bash
docker compose -f docker-compose.tripbot.yml up -d
docker compose -f docker-compose.website.yml up -d
docker compose -f docker-compose.wiki.yml up -d
```

---

## 🛡️ Step 5: Firewall Setup (UFW)

Ensure standard Ubuntu UFW rules allow required inbound ports:

```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow Web (Traefik)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow IRC (if running public Ergo IRC)
sudo ufw allow 6667/tcp
sudo ufw allow 6697/tcp

# Allow Wireguard VPN (if enabled)
sudo ufw allow 51820/udp

# Enable UFW
sudo ufw enable
sudo ufw status verbose
```

---

## 🔄 Updating & Restarting Services

To pull updated container images and restart a stack gracefully:

```bash
docker compose -f docker-compose.apps.yml pull
docker compose -f docker-compose.apps.yml up -d --remove-orphans
```
