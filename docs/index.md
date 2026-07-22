# TripSit Infrastructure Documentation

Welcome to the official infrastructure and server documentation for **TripSit Live**.

This documentation site is designed for systems administrators, operators, and developers who are familiar with standard **Ubuntu Linux** system administration but may be new to the TripSit server deployment and Docker architecture.

---

## 🚀 Quick Start & Navigation

Whether you are onboarding as a new sysadmin or looking up a specific service configuration, here is where to start:

- **[Architecture Overview](getting-started/overview.md)** — Understand the big picture: how network traffic flows, container isolation, and proxying work.
- **[Host Deployment Guide](getting-started/deployment.md)** — Step-by-step guide to preparing an Ubuntu server, configuring environment files, and launching services.
- **[Core Infrastructure](core-infrastructure/traefik.md)** — Deep dive into **Traefik**, **Docker Socket Proxy**, **CrowdSec**, **Fail2ban**, and **Portainer**.
- **[Services Catalog](services/chat-and-bridges.md)** — Browse all hosted services: Chat & Bridges (IRC, Matrix, Revcord), Web & Bots (Tripbot, Website, Calculators), Knowledge (MediaWiki, Wiki.js, Moodle), and Monitoring Utilities.
- **[Operations & Troubleshooting](operations/maintenance-and-troubleshooting.md)** — Day-to-day commands, logs, backups, and emergency recovery procedures.

---

## 🎯 Key Design Goals

The TripSit live server infrastructure is built with four primary tenets:

1. **Security-First Architecture**: Unrestricted socket exposure is disabled. All container access to Docker daemon is funneled through a fine-grained `docker-socket-proxy`. Intrusion prevention is enforced via Fail2ban, CrowdSec, and Traefik bouncers.
2. **Automated Reverse Proxy & SSL**: Traefik handles HTTP/HTTPS routing dynamically via Docker container labels and automatically provisions Let's Encrypt TLS certificates.
3. **Modular Composable Services**: Services are decoupled into logical `docker-compose.<name>.yml` files (e.g., `docker-compose.core.yml`, `docker-compose.tripbot.yml`, `docker-compose.matrix.yml`), allowing independent deployment and updates.
4. **Data Isolation**: Application runtime configurations, data databases, logs, and secrets are mapped cleanly to standard host paths under `/docker/`.

---

## 💡 Prerequisite Knowledge

To operate this environment efficiently, you should be comfortable with:
- Standard Ubuntu CLI commands (`systemctl`, `journalctl`, `chmod`/`chown`, `ufw`, `ss`/`netstat`).
- Basic Docker concepts (containers, images, volumes, bridge networks).
- Basic networking concepts (DNS records, A/AAAA records, TCP/UDP ports, reverse proxying).

No prior knowledge of TripSit's internal tools is required—every container and configuration file is explained step-by-step in this documentation!
