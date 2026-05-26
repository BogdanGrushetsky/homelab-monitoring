# Monitoring Stack — Glances + Prometheus + Grafana

## Stack

| Service | Port | Role |
|---|---|---|
| **Glances** | :7575 | Real-time web UI |
| **Node Exporter** | :7576 | System metrics collector |
| **Prometheus** | :7577 | Metrics storage (30-day retention) |
| **Grafana** | :7578 | Dashboards with historical data |

---

## Quick start

### 1. Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Temperature sensors (one-time setup)

```bash
sudo apt install lm-sensors -y
sudo sensors-detect --auto
sensors   # verify readings appear
```

### 3. Start

```bash
cd ~/Documents/monitoring
docker compose up -d
```

### 4. Open

| URL | What |
|---|---|
| http://localhost:7575 | Glances — real-time |
| http://localhost:7577 | Prometheus |
| http://localhost:7578 | Grafana — graphs + history |

Grafana login: `admin` / `admin` (change after first login)

---

## What you see in Grafana (Node Exporter Full dashboard)

- **CPU** — overall load, per-core, iowait, steal
- **Temperatures** — all hwmon sensors (CPU, motherboard, NVMe)
- **RAM** — used / cached / available over time
- **SWAP** — when and how much was used
- **Disks** — read/write IOPS, latency, space usage
- **Network** — traffic per interface
- **Load average** — 1m / 5m / 15m
- **Processes** — count and states

All graphs are browsable for **any period up to 30 days** — use the time range picker in the top-right corner of Grafana.

---

## Useful commands

```bash
# Check status of all services
docker compose ps

# Follow logs for a specific service
docker compose logs -f grafana
docker compose logs -f prometheus

# Restart all services
docker compose restart

# Update images
docker compose pull && docker compose up -d

# Stop (data is preserved in volumes)
docker compose stop

# Full teardown including all data
docker compose down -v
```

---

## Temperatures not showing

```bash
# Check if hwmon devices exist
ls /sys/class/hwmon/

# List sensor names
cat /sys/class/hwmon/hwmon*/name

# Load kernel modules manually
sudo modprobe coretemp      # Intel CPU
sudo modprobe k10temp       # AMD CPU
sudo modprobe thinkpad_acpi # ThinkPad

# Persist across reboots
echo "coretemp" | sudo tee -a /etc/modules
```

Then restart the collector: `docker compose restart node-exporter`

---

## Auto-start on boot

`restart: unless-stopped` is already set — everything starts automatically with Docker.

Make sure Docker itself starts on boot:

```bash
sudo systemctl enable docker
```

---

## Prometheus retention

Default is **30 days**. To change, edit `docker-compose.yml`:

```yaml
- '--storage.tsdb.retention.time=60d'  # 2 months
- '--storage.tsdb.retention.size=5GB'  # or by size limit
```

---

## File structure

```
.
├── docker-compose.yml
├── glances.conf
├── prometheus/
│   └── prometheus.yml
└── grafana/
    └── provisioning/
        ├── datasources/
        │   └── prometheus.yml
        └── dashboards/
            ├── dashboards.yml
            └── node-exporter.json   ← auto-provisioned
```
