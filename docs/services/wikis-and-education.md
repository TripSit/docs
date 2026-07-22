# Knowledge Bases & Educational Platforms

TripSit maintains multiple knowledge bases, community wikis, and an online learning management system (LMS) to curate drug education and harm reduction guides.

---

## 📚 1. TripSit MediaWiki (`docker-compose.wiki.yml`)

MediaWiki powers TripSit's primary public encyclopedia of substances, harm reduction guides, and safety protocols.

- **Stack Components**:
  - `wiki` / `mediawiki`: Runs **MediaWiki** (`bitnami/mediawiki:1.41.1` or standard `mediawiki:latest`).
  - `wiki_db`: **MariaDB** database storing wiki pages, revisions, user accounts, and category links.
- **Public Domain**: `https://wiki.tripsit.me`
- **Container Ports**:
  - Reverse proxied via Traefik.
  - Fallback internal host port: `8080:80`.

---

## 📖 2. Wiki.js & DrugWikiJS

TripSit operates **Wiki.js** instances for modern, markdown-based documentation and collaborative drug factsheets.

### Main Wiki.js Stack (`docker-compose.wikijs.yml`)
- **Application Container**: `wikijs` (`ghcr.io/requarks/wiki:2`)
- **Database**: `wikijs_db` (`postgres:15`)
- **Domain**: `https://wiki2.tripsit.me`

### DrugWikiJS Stack (`docker-compose.drugwikijs.yml`)
- **Application Container**: `drugwikijs` (`ghcr.io/requarks/wiki:2`)
- **Database**: `drugwikijs_db` (`postgres:14`)
- **Purpose**: Specialized factsheet repository focused specifically on chemical compounds, trip reports, and pharmacology metadata.

---

## 🎓 3. Moodle Learning Platform (`docker-compose.moodle.yml`)

TripSit operates **Moodle**, an open-source Learning Management System (LMS), to provide structured education courses for volunteers, trip-sitters, and community members.

- **Application Container**: `moodle` (`bitnami/moodle:4.5.3`)
- **Database Container**: `moodle_db_maria` (`bitnami/mariadb:11.3.2`)
- **Public Domain**: `https://learn.tripsit.me`
- **Data Persistence**: Mounts `/docker/appdata/moodle` and `/docker/appdata/moodledata`.

---

## 📰 4. WordPress Community Updates (`docker-compose.wordpress.yml`)

- **Application Container**: `wordpress` (`wordpress:latest`)
- **Database Container**: `wordpress_db` (`linuxserver/mariadb:latest`)
- **Public Domain**: `https://updates.tripsit.me`
- **Purpose**: News blog, announcement posts, and community updates.
