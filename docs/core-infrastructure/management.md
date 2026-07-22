# Container Management & CI/CD

This section details the management infrastructure, container maintenance, log monitoring, automated updates, backups, and continuous integration pipelines on TripSit Live.

---

## 🐳 1. Container Management (Portainer)

**Portainer** provides a GUI management console for monitoring container health, viewing container logs, inspecting volume allocations, and managing Docker networks.

- **Container Name**: `portainer`
- **Image**: `portainer/portainer-ee:latest`
- **Socket Access**: Connected via `socket-net` to `socket-proxy:2375`.
- **URL**: `https://portainer.tripsit.me` (Protected by Google OAuth SSO).

---

## 📋 2. Real-Time Log Viewer (Dozzle)

**Dozzle** is a lightweight web application for streaming real-time container logs without needing SSH access.

- **Container Name**: `dozzle`
- **Image**: `amir20/dozzle:latest`
- **Socket Access**: Connected to `socket-proxy:2375`.
- **URL**: `https://dozzle.tripsit.me` (Protected by Google OAuth SSO).

---

## 🔄 3. Automated Container Updates (Watchtower)

**Watchtower** monitors running Docker containers and automatically updates them when new images are published to registries.

### Selective Update Policy
To prevent unexpected breaking updates on production critical services, Watchtower updates are controlled explicitly via container labels:

```yaml
labels:
  # Enable automatic updates for this container
  - com.centurylinklabs.watchtower.enable=true

  # Disable automatic updates (e.g. for database or breaking state services)
  - com.centurylinklabs.watchtower.enable=false
```

---

## 🧹 4. Docker Garbage Collection (`dockergc`)

Over time, unused Docker images, dangling layers, and exited containers consume significant disk space.

- **Service**: `dockergc` (`clockworksoul/docker-gc-cron:latest`).
- **Function**: Runs on a automated cron schedule to prune dangling images, exited containers, and unused build caches.

---

## 💾 5. Backups & Disaster Recovery (Duplicati)

**Duplicati** manages automated, encrypted offsite backups for application configurations and databases.

- **Container Name**: `duplicati`
- **Image**: `ghcr.io/linuxserver/duplicati:latest`
- **Mounted Backup Sources**:
  - `/docker/appdata` (Application state)
  - `/docker/secrets` (Service credentials)
- **URL**: Protected behind Google OAuth SSO.

---

## 🚀 6. Continuous Integration & Deployment (Drone CI)

TripSit uses **Drone CI** for continuous integration and automated deployment builds.

- **Drone Server (`drone`)**: `drone/drone:2` — Handles webhooks, user permissions, and deployment triggers.
- **Drone Runner (`drone_runner`)**: `drone/drone-runner-docker:1` — Executes pipeline steps in isolated ephemeral Docker containers connected via `socket-proxy`.
