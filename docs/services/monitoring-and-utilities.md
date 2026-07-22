# Monitoring & Utility Services

This page documents the auxiliary tools, server performance monitoring systems, error tracking applications, and utility containers running on the server.

---

## 📊 1. Server Metrics & Monitoring (Netdata & Uptime Kuma)

### Netdata (`netdata`)
- **Image**: `netdata/netdata`
- **Purpose**: Real-time performance monitoring (CPU, RAM, disk I/O, network bandwidth, container utilization).
- **Security**: Access is protected behind Google OAuth SSO (`netdata.tripsit.me`).

### Uptime Kuma (`kuma`)
- **Image**: `louislam/uptime-kuma`
- **Purpose**: Uptime monitoring tool for HTTP endpoints, TCP ports, ping checks, and DNS records. Sends notifications (via Gotify / Discord) if a service goes down.
- **URL**: `kuma.tripsit.me`

---

## 🐞 2. Error Tracking & Quality (Glitchtip & SonarQube)

### Glitchtip Error Tracking (`docker-compose.glitchtip.yml`)
Glitchtip is an open-source, Sentry-compatible error logging application. It collects stack traces and runtime exception reports from Tripbot, websites, and custom microservices.

- **Web Service**: `glitchtip_web` (`glitchtip/glitchtip`, Port 8000).
- **Worker & Migration**: `glitchtip_worker`, `glitchtip_migrate`.
- **Database & Cache**: `glitchtip_postgres` (PostgreSQL 15), `glitchtip_redis` (Redis).

### SonarQube Code Analysis (`sonar`)
- **Image**: `sonarqube:community`
- **Database**: `sonar_db` (`bitnami/postgresql:15.4.0`)
- **Purpose**: Automated static code analysis, code quality metrics, and security vulnerability scanning for TripSit codebases.

---

## 📁 3. File Management & Remote Access (Filebrowser, SFTP, Wireguard)

### Filebrowser (`filebrowser`)
- **Image**: `filebrowser/filebrowser:s6`
- **Purpose**: Web-based interface to manage files within `/docker/appdata`.

### SFTP Service (`ftp`)
- **Image**: `atmoz/sftp:latest`
- **Exposed Host Port**: `2222:22`
- **Purpose**: Secure file transfer for authorized sysadmins.

### Wireguard VPN (`wireguard`)
- **Image**: `ghcr.io/wg-easy/wg-easy`
- **Exposed Host Ports**: `51820:51820/udp`, `51821:51821/tcp`
- **Purpose**: VPN server allowing administrators to securely tunnel into the internal host network.

---

## 🛠️ 4. Developer Tools & Documentation (IT-Tools & Trilium)

### IT-Tools (`it-tools`)
- **Image**: `corentinth/it-tools`
- **Purpose**: Collection of web-based developer utility tools (hash generators, regex testers, UUID generators, base64 converters).

### Trilium Notes (`trilium`)
- **Image**: `zadam/trilium:0.63.1-beta`
- **Purpose**: Hierarchical note-taking application for internal system architecture notes, runbooks, and operational guides.

---

## 🎮 5. Game & Community Dedicated Servers

### Stationeers Server (`stationeers`)
- **Compose File**: `docker-compose.stationeers.yml`
- **Ports**: `8443:8443`, `27015-27016` (UDP/TCP).

### Factorio Dedicated Server (`factorio`)
- **Compose File**: `docker-compose.apps.yml`
- **Ports**: `34197:34197/udp`, `27015:27015/tcp`.
