# SOC Lab — Elastic Stack (ELK) on macOS Apple Silicon

> A local SIEM lab built with Elasticsearch and Kibana using Docker Compose on an Apple Silicon MacBook Air (M1, 8GB RAM). Covers stack setup, authentication configuration, service connectivity, Kibana access, and documented troubleshooting.

---

## Objectives

- Deploy Elasticsearch and Kibana locally with Docker Compose
- Configure secure inter-service authentication between Kibana and Elasticsearch
- Validate Kibana browser access and login
- Understand and debug common Elastic startup and connectivity issues
- Document a reproducible setup process for portfolio and future reference

---

## Lab Architecture

```
┌─────────────────────────────────────────┐
│         macOS Host (M1, 8GB RAM)        │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │        Docker Network           │    │
│  │                                 │    │
│  │  ┌──────────────┐               │    │
│  │  │ Elasticsearch│  :9200        │    │
│  │  │    es01      │               │    │
│  │  └──────┬───────┘               │    │
│  │         │ http://elasticsearch  │    │
│  │  ┌──────▼───────┐               │    │
│  │  │    Kibana    │  :5601        │    │
│  │  │    kib01     │               │    │
│  │  └──────────────┘               │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

| Component     | Version | Port | Container  |
|---------------|---------|------|------------|
| Elasticsearch | 8.17.4  | 9200 | es01       |
| Kibana        | 8.17.4  | 5601 | kib01      |

---

## Tech Stack

- Docker Desktop (Apple Silicon)
- Elasticsearch 8.17.4
- Kibana 8.17.4
- macOS Terminal / zsh
- curl

---

## Prerequisites

- Docker Desktop installed and running
- At least 3GB RAM allocated to Docker
- Ports 9200 and 5601 free on the host

---

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/obaidadil/soc-lab-elk-stack.git
cd soc-lab-elk-stack
```

### 2. Start the stack

```bash
docker compose up -d
```

### 3. Wait for Elasticsearch to become healthy

```bash
docker compose ps
```

Elasticsearch has a healthcheck configured. Kibana will not start until it passes.

### 4. Reset the kibana_system password

Once Elasticsearch is healthy, set the `kibana_system` password to match what is in `docker-compose.yml`:

```bash
docker exec -it es01 \
  elasticsearch-reset-password -u kibana_system -i
```

Enter `KibanaSystem123!` when prompted.

### 5. Verify Elasticsearch authentication

```bash
curl -s -u 'kibana_system:KibanaSystem123!' \
  http://localhost:9200/_security/_authenticate?pretty
```

Expected: a JSON response with `"username" : "kibana_system"`.

### 6. Check Kibana is up

```bash
curl -I http://127.0.0.1:5601
```

Expected: `HTTP/1.1 302 Found`

### 7. Open Kibana in the browser

```
http://localhost:5601
```

Login with:
- **Username:** `elastic`
- **Password:** `ChangeThisNow123!`

---

## Key Configuration Notes

### Docker service naming matters

Kibana connects to Elasticsearch using the **Docker service name**, not the container name. In `docker-compose.yml`:

```yaml
# CORRECT — uses the service name defined under `services:`
ELASTICSEARCH_HOSTS=http://elasticsearch:9200

# WRONG — es01 is the container name, not the service name
ELASTICSEARCH_HOSTS=http://es01:9200
```

### Two separate passwords

| Account         | Used for                          | Password           |
|-----------------|-----------------------------------|--------------------|
| `elastic`       | Browser login to Kibana           | `ChangeThisNow123!`|
| `kibana_system` | Kibana-to-Elasticsearch API calls | `KibanaSystem123!` |

These are different accounts. `kibana_system` cannot be used to log in to the Kibana UI.

### Memory limits

On an M1 MacBook Air with 8GB RAM, keep JVM heap conservative:

```yaml
ES_JAVA_OPTS=-Xms1g -Xmx1g
```

Allocate at least 3GB total to Docker in Docker Desktop settings.

---

## Issues Encountered

### 1. Wrong Elasticsearch host in Kibana config
**Problem:** Used `http://es01:9200` (container name) instead of `http://elasticsearch:9200` (service name).  
**Fix:** Changed the `ELASTICSEARCH_HOSTS` value in the Kibana service block to use the service name.

### 2. Kibana credentials mismatch
**Problem:** `kibana_system` password in Docker Compose did not match the password set inside Elasticsearch, causing repeated `security_exception` errors.  
**Fix:** Used `elasticsearch-reset-password` inside the container to force the password to match.

### 3. Misplaced environment variables
**Problem:** Kibana-specific env vars (`ELASTICSEARCH_HOSTS`, `SERVER_HOST`) were accidentally placed inside the Elasticsearch service block.  
**Fix:** Moved them to the correct Kibana service block.

### 4. zsh special character interference
**Problem:** Certain characters in commands were interpreted by zsh before reaching the shell, breaking curl and YAML syntax.  
**Fix:** Used single quotes around curl credentials and passwords.

### 5. Kibana showing empty Discover / "Add integrations"
**Problem:** Discover was empty after login because no data had been ingested and no data view existed.  
**Status:** Expected behavior with a fresh stack. Sample data was loaded through Kibana Home to verify the UI worked correctly.

---

## Lessons Learned

- Docker Compose service names and container names are not the same thing — inter-container communication uses service names
- Elasticsearch 8.x security is enabled by default; credential mismatches fail silently in Kibana until you check logs
- Always validate Elasticsearch auth directly with curl before debugging Kibana
- `kibana_system` is a backend service account only — not for browser login
- Local labs on low-memory hardware need tight JVM and Docker memory limits or containers will OOM silently
- zsh handles special characters differently than bash — quote everything in curl commands

---

## Project Status

The stack was successfully deployed and Kibana login was verified. The project was intentionally scoped to the infrastructure and authentication layer only. Full log ingestion, detection rules, and dashboards were not implemented in this iteration due to local hardware constraints.

The next iteration of this lab moves to **Azure + Splunk** for a cloud-based pipeline with real log sources, detection rules, and alerting.

---

## Next Project

[Azure + Splunk SIEM Lab](https://github.com/obaidadil) — Cloud-based SOC lab with Azure log forwarding, Splunk ingestion, dashboards, and detection alerts.

---

## Screenshots

See the [`screenshots/`](./screenshots) directory.

---

## Author

**Obaid Adil** — IT Operations Analyst transitioning into Cybersecurity  
[GitHub](https://github.com/obaidadil)
