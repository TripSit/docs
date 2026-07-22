# Traefik Reverse Proxy & DNS Integration

This section explains how **Traefik** acts as the central ingress router for all web traffic targeting the TripSit live server infrastructure.

---

## 🎯 Role of Traefik

Traefik is a modern edge router / reverse proxy that automatically discovers containers running on Docker networks and routes HTTP/HTTPS requests to them based on dynamic configuration labels.

Key benefits in TripSit's architecture:
- **No Manual Nginx/Apache Routing Files**: Service routes are defined directly in each service's `docker-compose.<name>.yml` using container labels.
- **Automated SSL/TLS Certificates**: Obtains and automatically renews SSL certificates from **Let's Encrypt** via DNS-01 or HTTP-01 challenges.
- **Cloudflare Companion Integration**: Automatically creates and updates Cloudflare DNS records when new containers with routing labels are created.

---

## ⚙️ Core Traefik Service Breakdown

In `docker-compose.core.yml`:

```yaml
  traefik:
    image: traefik:latest
    container_name: traefik
    ports:
      - "80:80"     # HTTP entrypoint (redirects to 443)
      - "443:443"   # HTTPS entrypoint
    networks:
      - traefik-net
      - socket-net
    environment:
      - CF_API_EMAIL=${CLOUDFLARE_EMAIL}
      - CF_API_KEY=${CLOUDFLARE_API_KEY}
    volumes:
      - ${DOCKERDIR}/traefik/acme.json:/acme.json
      - ${DOCKERDIR}/traefik/traefik.yml:/etc/traefik/traefik.yml:ro
      - ${DOCKERDIR}/traefik/rules:/rules:ro
```

---

## 🏷️ How Traefik Container Labels Work

When adding a new web container to any compose file, Traefik routes traffic to it using standard labels.

### Standard Label Template:
```yaml
labels:
  - com.centurylinklabs.watchtower.enable=true
  - traefik.enable=true
  - traefik.docker.network=traefik-net
  - traefik.http.routers.<service_name>.rule=Host(`subdomain.${DNS_DOMAIN}`)
  - traefik.http.routers.<service_name>.entrypoints=websecure
  - traefik.http.routers.<service_name>.tls.certresolver=letsencrypt
  - traefik.http.services.<service_name>-svc.loadbalancer.server.port=8080
```

### Explanation of Key Labels:
- `traefik.enable=true`: Tells Traefik to expose this container.
- `traefik.docker.network=traefik-net`: Specifies which network Traefik should use to reach the container.
- `traefik.http.routers.<service_name>.rule`: Defines the rule for matching incoming HTTP requests (e.g. Host matching, path prefix matching).
- `traefik.http.routers.<service_name>.entrypoints`: Specifies entrypoint (`websecure` for HTTPS / port 443).
- `traefik.http.routers.<service_name>.tls.certresolver`: Directs Traefik to use the configured Let's Encrypt certificate resolver.
- `traefik.http.services.<service_name>-svc.loadbalancer.server.port`: Explicitly declares which container internal port receives traffic.

---

## 🌐 Cloudflare Companion (`cloudflare_companion`)

In `docker-compose.core.yml`:
```yaml
  cloudflare_companion:
    image: tiredofit/traefik-cloudflare-companion:latest
    container_name: cloudflare_companion
    environment:
      - TIMEZONE=${TZ}
      - TRAEFIK_VERSION=2
      - CF_EMAIL=${CLOUDFLARE_EMAIL}
      - CF_TOKEN=${CLOUDFLARE_API_KEY}
      - TARGET_DOMAIN=${DNS_DOMAIN}
      - DOMAIN1=${DNS_DOMAIN}
```

### How Cloudflare Companion Works:
1. Listens to Docker container state events via `socket-proxy`.
2. Inspects `Host(...)` rules in new container labels.
3. Automatically creates a CNAME or A record in Cloudflare pointing to the server's public IP address.
4. Removes DNS records when containers are destroyed.

---

## 🔑 Traefik SSL Exporter (`ssl_export`)

In `docker-compose.irc.yml`:
```yaml
  ssl_export:
    image: rafi0101/traefik-ssl-certificate-exporter
    container_name: ssl_export
    volumes:
      - ${DOCKERDIR}/traefik/acme.json:/acme.json:ro
      - ${DOCKERDIR}/certs:/output:rw
```

### Purpose:
Traefik stores certificates in a single JSON file (`acme.json`). Non-Traefik native services—such as the **Ergo IRC daemon**—require standard `.crt` and `.key` PEM files on disk.

`ssl_export` monitors `acme.json` and automatically extracts certificates into standard PEM files inside `${DOCKERDIR}/certs/` whenever Traefik renews a certificate.
