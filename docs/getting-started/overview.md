# Architecture Overview

This page provides a high-level conceptual overview of the TripSit live server environment, networking layout, and security architecture.

---

## 🏗️ High-Level System Architecture

TripSit Live operates on a modular **Docker Compose** stack running on Ubuntu Linux. Rather than putting every service into a single monolithic compose file, the infrastructure is split across targeted stack files.

```
                  ┌──────────────────────────────────────────────┐
                  │                 Internet                     │
                  └──────────────────────┬───────────────────────┘
                                         │
                                   Ports 80 / 443
                                         │
                                         ▼
                  ┌──────────────────────────────────────────────┐
                  │           Traefik Reverse Proxy              │
                  │   (Automatic TLS via Let's Encrypt / DNS)    │
                  └━━━━━━┬━━━━━━━━━━━━━━━┬━━━━━━━━━━━━━━━┬━━━━━━━┘
                         │               │               │
      ┌──────────────────┴──┐  ┌─────────┴─────────┐  ┌──┴────────────────┐
      │ Core Infrastructure │  │   Chat & Bridges  │  │  Web, Bots & Apps  │
      │  - Socket Proxy     │  │  - Ergo IRC       │  │  - Tripbot API     │
      │  - CrowdSec / F2B   │  │  - Synapse Matrix │  │  - Main Website    │
      │  - OAuth / Keycloak │  │  - Element Web    │  │  - MediaWiki       │
      │  - Portainer        │  │  - Revcord Bridge │  │  - Calculators     │
      └─────────────────────┘  └───────────────────┘  └────────────────────┘
```

---

## 🌐 Network Flow & Ingress

### 1. External Traffic Entry
- **Port 80 (HTTP)**: Traffic on HTTP is automatically redirected to HTTPS (Port 443) by Traefik.
- **Port 443 (HTTPS)**: All web services are routed through Traefik using **Host-based routing labels** defined on individual containers (e.g. `Host(\`wiki.tripsit.me\`)`).
- **Direct Service Ports**: Select non-HTTP services expose dedicated host ports (e.g., SSH on port 22, Ergo IRC on 6667/6697, Stalwart Mail on 25/587/465/143/993, Wireguard on 51820 UDP).

### 2. Internal Docker Networks
Every compose stack defines isolated internal docker networks (e.g., `traefik-net`, `socket-net`, `tripbot_database-net`, `matrix_db-net`).
- **Network Isolation**: Backend databases (PostgreSQL, MariaDB) exist on private networks accessible *only* by their respective application containers.
- **Proxy Network (`traefik-net`)**: Traefik joins the `traefik-net` network to forward incoming web requests to application containers without exposing internal application ports directly to the host host interface.

---

## 🔒 Security Principles

### 1. Docker Socket Proxy (`socket-proxy`)
!!! warning "Critical Security Requirement"
    Directly mounting the Docker socket (`/var/run/docker.sock`) into containers gives root-equivalent host privileges.

To eliminate this vulnerability:
- The host Unix socket is disabled for standard application containers.
- A hardened **`tecnativa/docker-socket-proxy`** container listens on an isolated internal network (`socket-net`).
- Containers like **Traefik**, **Portainer**, and **Watchtower** query `socket-proxy` with strict read-only or minimal permission flags set via environment variables.

### 2. Intrusion Detection & Prevention
- **Fail2ban**: Monitors log files for brute-force attacks and dynamically updates firewall rules.
- **CrowdSec**: Analyzes ingress web logs via Traefik bouncers to block known malicious IP addresses collaboratively across community threat intelligence.
- **Traefik Forward Auth (Google OAuth)**: Protects administrative dashboards (Portainer, Dozzle, Netdata, Duplicati) behind Google Single Sign-On (SSO).

---

## 📁 Storage & Host Layout

All persistent application data, custom Dockerfiles, logs, and secrets reside under a standardized host hierarchy:

| Host Directory | Description | Access Permission |
| :--- | :--- | :--- |
| `/docker/appdata/` | Persistent volumes for application state (databases, configs, uploads) | Service-specific |
| `/docker/custom/` | Custom `Dockerfile` build contexts for TripSit apps | Root / Operator |
| `/docker/logs/` | Centralized log folder mounted by containers | Read/Write |
| `/docker/secrets/` | Credentials, API tokens, and certificate files | Root only (0600/0700) |
| `/docker/scripts/` | Maintenance, maintenance automation, and health check scripts | Operator |

---

## 📦 Stack Breakdown

The system is organized into distinct compose files:

- **`docker-compose.core.yml`**: Traefik, Socket Proxy, CrowdSec, Fail2ban, OAuth, Portainer, Duplicati, Drone, Keycloak.
- **`docker-compose.apps.yml`**: Admin utilities (Dozzle, Netdata, Filebrowser, IT-tools, Uptime Kuma, Wireguard, SonarQube, Trilium).
- **`docker-compose.tripbot.yml`**: Tripbot discord bot, production API, database, pgAdmin, and staging deployment.
- **`docker-compose.website.yml`**: Main TripSit website container stack.
- **`docker-compose.irc.yml`**: Ergo IRC server and Traefik SSL exporter.
- **`docker-compose.matrix.yml`**: Synapse Matrix homeserver, Element Web client, Matrix Admin, Coturn TURN server, Redis, Postgres.
- **`docker-compose.wiki.yml` / `wikijs.yml` / `drugwikijs.yml`**: Knowledge base mediawiki & Wiki.js stacks.
- **`docker-compose.moodle.yml`**: Moodle learning platform (`learn.tripsit.me`).
- **`docker-compose.stalwart.yml`**: Stalwart mail server.
