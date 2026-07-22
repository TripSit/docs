# Maintenance & Troubleshooting Guide

This guide provides practical administrative procedures, useful shell commands, and troubleshooting workflows for managing TripSit Live.

---

## 🛠️ Essential Management Commands

All container management should be performed from the root directory of the repository (`/home/moonbear/UAT`) where compose files and `.env` live.

### 1. View Service Status
```bash
# View status of core services
docker compose -f docker-compose.core.yml ps

# View all containers running on the host
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### 2. View Real-Time Logs
```bash
# Follow logs for Traefik
docker compose -f docker-compose.core.yml logs -f traefik

# Follow logs for Tripbot
docker compose -f docker-compose.tripbot.yml logs -f tripbot

# Follow logs for a specific container by name
docker logs -f --tail 100 matrix_synapse
```

### 3. Restart a Specific Service Stack
```bash
# Restart core stack without affecting other stacks
docker compose -f docker-compose.core.yml restart

# Force recreate a single container after updating configuration
docker compose -f docker-compose.tripbot.yml up -d --force-recreate tripbot
```

---

## 🔄 Updating Stack Services

To update services safely to their latest docker images:

```bash
# Step 1: Pull updated images for target stack
docker compose -f docker-compose.apps.yml pull

# Step 2: Recreate containers with new images (downtime < 2 seconds)
docker compose -f docker-compose.apps.yml up -d --remove-orphans

# Step 3: Prune unused old image layers
docker image prune -f
```

---

## 🔍 Common Troubleshooting Scenarios

### Scenario A: Traefik Returns "404 Not Found" for a Domain
1. **Verify Dynamic Routing Label**: Ensure container has `traefik.enable=true` and `traefik.http.routers.<name>.rule=Host(...)` labels.
2. **Check Container Network**: Container MUST be connected to `traefik-net`.
3. **Inspect Traefik Dashboard / Logs**:
   ```bash
   docker logs traefik --tail 50 | grep -i error
   ```

### Scenario B: Database Connection Errors in Container
1. **Check Database Network**: Application containers must share a common docker network with their database container (e.g. `moodle-net` & `moodle_data-net`).
2. **Verify Environment Passwords**: Confirm `.env` database passwords match application config files.
3. **Check Database Container Status**:
   ```bash
   docker ps | grep db
   docker logs tripbot_database --tail 50
   ```

### Scenario C: Socket Proxy Permission Denied Errors
If Portainer, Watchtower, or Traefik complain about Docker API access:
1. Verify `docker-socket-proxy` container is running (`docker ps | grep socket-proxy`).
2. Verify target container environment variable `DOCKER_HOST` is set to `tcp://socket-proxy:2375`.
3. Check `docker-socket-proxy` environment flags in `docker-compose.core.yml` to confirm the required endpoint (e.g. `CONTAINERS: 1`, `POST: 1`) is granted (`1`).

---

## 💾 Backup & Restore Procedures

### Database Backups (PostgreSQL & MariaDB)

#### 1. Manual Backup of PostgreSQL (e.g., Tripbot Database)
```bash
docker exec -t tripbot_database pg_dump -U postgres tripbot > /docker/appdata/tripbot/tripbot_backup_$(date +%F).sql
```

#### 2. Manual Restore of PostgreSQL
```bash
cat /docker/appdata/tripbot/tripbot_backup.sql | docker exec -i tripbot_database psql -U postgres -d tripbot
```

#### 3. Manual Backup of MariaDB (e.g., MediaWiki Database)
```bash
docker exec -t wiki_db mysqldump -u root -p"${MARIADB_ROOT_PASSWORD}" wiki > /docker/appdata/wiki/wiki_backup_$(date +%F).sql
```

---

## 🧹 System Cleanup & Disk Maintenance

Check disk usage across Docker containers, images, and volumes:
```bash
docker system df
```

Prune stopped containers, unused networks, and dangling images:
```bash
docker system prune -f
```

Prune unused data volumes (**Use with caution—ensure no persistent data volumes are orphan!**):
```bash
docker volume prune
```
