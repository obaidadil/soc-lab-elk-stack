# SOC Home Lab — ELK Stack + Filebeat

A home security operations center (SOC) lab built on a MacBook Air M1 (8GB RAM) using Docker and VirtualBox. Demonstrates end-to-end log ingestion from a virtual Ubuntu machine into a self-hosted Elastic SIEM stack.

## Architecture

```
Ubuntu VM (victim-server)
        |
    Filebeat 8.19.14
    (journald input)
        |
        v
Mac Host — Docker Desktop
  ├── Elasticsearch 8.17.4  :9200
  └── Kibana 8.17.4         :5601
```

- The Ubuntu VM runs in VirtualBox using NAT networking
- From the VM, the Mac host is reachable at `10.0.2.2`
- Filebeat reads from `journald` and ships logs directly to Elasticsearch
- Kibana provides the search and visualization interface via `http://localhost:5601`

## Stack

| Component       | Version  | Role                        |
|----------------|----------|-----------------------------|
| Elasticsearch  | 8.17.4   | Log storage and indexing    |
| Kibana         | 8.17.4   | Visualization and search    |
| Filebeat       | 8.19.14  | Log shipper (Ubuntu VM)     |
| Docker Desktop | Latest   | Container runtime (Mac)     |
| VirtualBox     | Latest   | Ubuntu VM hypervisor        |

## Prerequisites

- MacBook (Apple Silicon or Intel) with Docker Desktop installed
- VirtualBox with an Ubuntu Server VM (NAT networking)
- VM must be able to reach Mac host at `10.0.2.2`

## Setup

### 1. Start the SIEM stack on the Mac

```bash
cd ~/soc-lab
docker compose up -d
```

### 2. Set the Kibana system password

```bash
curl -u elastic:YOUR_PASSWORD \
  -X POST http://localhost:9200/_security/user/kibana_system/_password \
  -H 'Content-Type: application/json' \
  -d '{"password":"YOUR_KIBANA_PASSWORD"}'
```

### 3. Install Filebeat on the Ubuntu VM

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | \
  sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] \
  https://artifacts.elastic.co/packages/8.x/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update && sudo apt install filebeat -y
```

### 4. Configure Filebeat

See `filebeat/filebeat.yml` for the full configuration.

Key settings:
- Input type: `journald` (reads systemd journal directly)
- Output: Elasticsearch at `10.0.2.2:9200`
- Authentication: username/password

```bash
sudo systemctl enable filebeat
sudo systemctl restart filebeat
```

### 5. Verify in Kibana

1. Open `http://localhost:5601`
2. Go to **Discover**
3. Select the `filebeat-*` Data View
4. Set time range to **Last 15 minutes**
5. Confirm log documents are appearing from `victim-server`

## Results

- 113+ log documents ingested from Ubuntu VM in the first session
- Logs visible in Kibana Discover under the `filebeat-*` data view
- Key fields captured: `host.hostname`, `agent.type`, `event.created`, `ecs.version`, `data_stream.dataset`

## Skills Demonstrated

- SIEM deployment and configuration (Elastic Stack)
- Log ingestion pipeline design (Filebeat → Elasticsearch)
- Docker Compose for security tooling on constrained hardware
- Elasticsearch index and data stream management
- Kibana data view creation and log analysis
- Linux log management using `journald`
- VirtualBox NAT networking for isolated lab environments
- Security hardening: xpack.security enabled with authentication

## Project Status

- [x] Elasticsearch + Kibana running in Docker
- [x] Ubuntu VM shipping logs via Filebeat
- [x] Logs visible and searchable in Kibana
- [ ] Add Logstash for log parsing pipelines
- [ ] Add Windows VM as second log source
- [ ] Build detection rules in Kibana Security
- [ ] Simulate attack scenarios (brute force, privilege escalation)
