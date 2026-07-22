# Bots & Web Applications

This section documents TripSit's core application ecosystem, including the flagship **Tripbot** service, the main web portal, and harm reduction tools.

---

## 🤖 1. Tripbot Ecosystem (`docker-compose.tripbot.yml`)

**Tripbot** is TripSit's primary Discord bot and harm-reduction assistant. It provides drug interaction lookup, dosage information, harm-reduction guides, and community support workflows.

```
                         ┌──────────────────────────┐
                         │   Discord / Web Clients  │
                         └────────────┬─────────────┘
                                      │
                                HTTPS / API
                                      │
                                      ▼
┌───────────────────────┐   ┌───────────────────┐   ┌───────────────────────┐
│ Tripbot Bot & API     │──►│ PostgreSQL DB     │◄──│ pgAdmin4 Dashboard    │
│ (container: tripbot)  │   │ (tripbot_database)│   │ (tripbot_pgadmin)     │
└───────────────────────┘   └───────────────────┘   └───────────────────────┘
```

### Components Breakdown

| Container Name | Image / Context | Purpose & Routing |
| :--- | :--- | :--- |
| `tripbot` | `custom/build` | Core Tripbot bot instance and public REST API server (`api.tripsit.me`). |
| `tripbot_database` | `postgres:15.4` | Production PostgreSQL database storing user preferences, bot settings, and logs. |
| `tripbot_deploy` | `custom/build` | Staging/development deployment runner (`tripbot-deploy.tripsit.me`). |
| `tripbot_database_deploy` | `postgres:15.4` | Staging PostgreSQL database. |
| `tripbot_pgadmin` | `dpage/pgadmin4:latest` | Web-based database management interface (`database.tripsit.me`, `pgadmin.tripsit.me`). |

---

## 🌐 2. Main TripSit Website (`docker-compose.website.yml`)

- **Container Name**: `website`
- **Build Context**: Custom build (`/docker/custom/website` or workspace repository).
- **Networks**: `website-net` connected to Traefik.
- **Routing Rules**:
  - `Host(\`tripsit.me\`)`
  - `Host(\`chat.tripsit.me\`)`
  - `Host(\`drugs.tripsit.me\`)`
  - `Host(\`support.tripsit.me\`)`

The main web application serves public harm reduction resources, drug factsheet search interfaces, and direct links to IRC and Matrix chat rooms.

---

## 🧮 3. Harm Reduction Calculators (`docker-compose.apps.yml`)

TripSit hosts standalone interactive calculators to assist users in dosage safety and volumetric measurement:

### Benzodiazepine Equivalence Calculator (`calculator_benzos`)
- **Container Name**: `calculator_benzos`
- **Build Context**: `custom/build`
- **Purpose**: Calculates dosage equivalencies across common benzodiazepine compounds to reduce accidental overdose risks.

### Volumetric Liquid Dosing Calculator (`calculator_volume`)
- **Container Name**: `calculator_volume`
- **Build Context**: `custom/build`
- **Purpose**: Calculates precise liquid volume concentrations for accurate micro-dosing and volumetric administration safety.
