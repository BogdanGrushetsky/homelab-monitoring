# 🖥️ Laptop / Server Monitoring Stack

A self-hosted monitoring stack for tracking system resources 24/7 — CPU, RAM, temperatures, disks, network — with 30 days of historical data.

## Stack

| Service | Port | Role |
|---|---|---|
| [Glances](https://github.com/nicolargo/glances) | `7575` | Real-time web UI |
| [Node Exporter](https://github.com/prometheus/node_exporter) | `7576` | System metrics collector |
| [Prometheus](https://prometheus.io) | `7577` | Time-series storage (30d retention) |
| [Grafana](https://grafana.com) | `7578` | Dashboards with historical data |

## What's monitored

- **CPU** — overall load, per-core, iowait
- **RAM & SWAP** — usage over time
- **Temperatures** — all hwmon sensors (CPU, motherboard, NVMe)
- **Disks** — I/O, latency, space usage per partition
- **Network** — traffic per interface
- **Load average** — 1m / 5m / 15m
- **Processes** — count and states
- **Battery** — charge level (laptops)
- **Docker containers** — resource usage

## Requirements

- Linux (Ubuntu / Debian / Arch)
- Docker Engine 20.10+ (install via `get.docker.com`, not via snap)
- Docker Compose v2

> **Note:** The snap version of Docker has kernel namespace restrictions that prevent `pid: host` and `rslave` bind mounts. Use the official Docker Engine package.

## Quick start

### 1. Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Enable temperature sensors

```bash
sudo apt install lm-sensors -y
sudo sensors-detect --auto
sensors  # verify readings appear
```

> For Intel CPUs: `sudo modprobe coretemp`  
> For AMD CPUs: `sudo modprobe k10temp`  
> For ThinkPads: `sudo modprobe thinkpad_acpi`

To persist across reboots:
```bash
echo "coretemp" | sudo tee -a /etc/modules
```

### 3. Clone and start

```bash
git clone https://github.com/<your-username>/monitoring.git
cd monitoring
docker compose up -d
```

### 4. Open dashboards

| URL | What |
|---|---|
| http://localhost:7575 | Glances — real-time overview |
| http://localhost:7578 | Grafana — historical graphs |

Grafana default login: `admin` / `admin`

The **Node Exporter Full** dashboard is provisioned automatically — no manual import needed.

## Configuration

### Alert thresholds (`glances.conf`)

Thresholds use a three-level color system: blue (careful) → yellow (warning) → red (critical).

```
CPU:          60% / 80% / 95%
RAM:          60% / 80% / 95%
Temperature:  60°C / 75°C / 90°C
Disk space:   50% / 70% / 90%
Battery:      20% / 10% / 5%
```

Edit [`glances.conf`](glances.conf) to adjust any threshold.

### Data retention

Default is **30 days**. Change in `docker-compose.yml`:

```yaml
- '--storage.tsdb.retention.time=60d'  # by time
- '--storage.tsdb.retention.size=5GB'  # or by size
```

## Auto-start on boot

The `restart: unless-stopped` policy is already set for all containers. Make sure Docker starts on boot:

```bash
sudo systemctl enable docker
```

## Useful commands

```bash
# Check status
docker compose ps

# View logs
docker compose logs -f grafana

# Update images
docker compose pull && docker compose up -d

# Stop (data is preserved in volumes)
docker compose stop

# Full teardown including data
docker compose down -v
```

## Directory structure

```
.
├── docker-compose.yml
├── glances.conf
├── prometheus/
│   └── prometheus.yml
└── grafana/
    └── provisioning/
        ├── datasources/
        │   └── prometheus.yml      # auto-configures Prometheus datasource
        └── dashboards/
            ├── dashboards.yml
            └── node-exporter.json  # Node Exporter Full dashboard
```

## License

MIT
