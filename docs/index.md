# Documentation archived

The maintained operational documentation moved to [TripSit/live](https://github.com/TripSit/live/tree/main/docs), a private repository requiring TripSit infrastructure access:

- `docs/architecture.md`
- `docs/networking.md`
- `docs/storage-and-backups.md`
- `docs/troubleshooting.md`

The migration was verified on 16 November 2026 and saved in [live commit f54111b](https://github.com/TripSit/live/commit/f54111bcc62e5e5f8e2b7afe442acb2cb9b4e84f). Runtime Docker inspection and restoration remain explicitly unverified; the maintained documents record the checks actually performed.

All original pages, site configuration, workflows, and Git history are preserved here. The historical pages below describe an older setup and must not be used as current operational instructions. Submit future infrastructure documentation changes to TripSit/live.

---

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
