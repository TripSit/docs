# Security & Access Control

This page covers the security mechanisms deployed across the TripSit server stack to protect infrastructure, isolate containers, detect threats, and enforce Single Sign-On (SSO).

---

## 🔒 1. Docker Socket Proxy (`docker-socket-proxy`)

The Docker UNIX socket (`/var/run/docker.sock`) is the control plane for the Docker daemon. Providing unconfined access to `/var/run/docker.sock` allows any container to gain root access to the entire host machine.

### Security Architecture
TripSit uses `tecnativa/docker-socket-proxy`, an HAProxy-based security sidecar, to filter and restrict access to the Docker API.

```
┌──────────────────┐               ┌─────────────────────┐               ┌─────────────────┐
│ Container        │ API Request   │ docker-socket-proxy │ Read-only Unix  │ Docker Daemon   │
│ (Portainer, etc.)│──────────────►│ (TCP 127.0.0.1:2375)│────────────────►│ (/docker.sock)  │
└──────────────────┘               └─────────────────────┘               └─────────────────┘
                                              │
                                    Blocks Auth/Exec/Secrets
```

### Configuration Rules (`docker-compose.core.yml`)

```yaml
  docker-socket-proxy:
    image: tecnativa/docker-socket-proxy
    container_name: socket-proxy
    ports:
      - "127.0.0.1:2375:2375" # Exposed ONLY to local loopback/internal bridge network
    networks:
      - socket-net
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      # Granted permissions
      EVENTS: 1
      PING: 1
      VERSION: 1
      CONTAINERS: 1
      IMAGES: 1
      NETWORKS: 1
      SERVICES: 1
      VOLUMES: 1
      POST: 1       # Required by Watchtower
      BUILD: 1      # Required by Drone CI
      # Strictly revoked endpoints
      AUTH: 0
      SECRETS: 0
      COMMIT: 0
      CONFIGS: 0
      EXEC: 0       # Prevents arbitrary container execution via socket
      NODES: 0
      SWARM: 0
```

---

## 🛡️ 2. Intrusion Detection & Prevention (CrowdSec & Fail2ban)

### Fail2ban (`fail2ban`)
Fail2ban scans system and container logs for brute-force patterns and dynamically manipulates host iptables firewall rules to drop offensive IPs.

- **Image**: `crazymax/fail2ban:latest`
- **Network Mode**: `host` with `NET_ADMIN` and `NET_RAW` Linux capabilities.
- **Monitored Logs**: `/docker/logs/fail2ban`, `/docker/logs/traefik`.

### CrowdSec IPS & Traefik Bouncer (`crowdsec`, `crowdsec_traefik`)
CrowdSec is an open-source collaborative Intrusion Prevention System (IPS).

- **Database**: Dedicated MariaDB container (`crowdsec_db`).
- **Collections**: Configured with rule collections for Traefik, HTTP CVEs, Linux system logs, and iptables.
- **Traefik Bouncer**: `crowdsec_traefik` integrates directly with Traefik as middleware. When CrowdSec flags an offending IP address globally or locally, the bouncer blocks the HTTP request at the Traefik router layer before it reaches any backend container.

---

## 🔑 3. Google OAuth Single Sign-On (`oauth`)

Administrative web tools (Portainer, Dozzle logs, Netdata metrics, Duplicati backups) are protected behind Google OAuth SSO via `thomseddon/traefik-forward-auth`.

### How OAuth Forward Auth Middleware Works
1. An unauthenticated user attempts to visit `https://portainer.tripsit.me`.
2. Traefik checks the container middleware label `googleAuth@file`.
3. Traefik redirects the browser to `https://oauth.tripsit.me/_oauth` for Google authentication.
4. If the user's email address is present in the `OAUTH_WHITELIST` environment variable, OAuth sets a secure encrypted cookie and grants access.
5. If unauthorized, access is denied.

```yaml
  oauth:
    image: thomseddon/traefik-forward-auth:latest
    container_name: oauth
    environment:
      - CLIENT_ID=${GOOGLE_CLIENT_ID}
      - CLIENT_SECRET=${GOOGLE_CLIENT_SECRET}
      - SECRET=${OAUTH_SECRET}
      - COOKIE_DOMAIN=${DNS_DOMAIN}
      - WHITELIST=${OAUTH_WHITELIST}
      - DEFAULT_ACTION=auth
      - DEFAULT_PROVIDER=google
```

---

## 🔐 4. Keycloak Identity Management (`keycloak`)

For advanced user identity management and single sign-on across TripSit platforms:
- **Service**: `keycloak` (`docker.io/bitnami/keycloak:21.1.2`).
- **Database**: `keycloak_db` (PostgreSQL 15.4).
- **Network**: Private `keycloak_db-net` network isolated from public access.
