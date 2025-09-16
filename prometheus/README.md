# Ubuntu Monitoring Stack: Prometheus, Grafana, Loki, Node Exporter, Promtail

This guide helps you set up a complete monitoring stack on any Ubuntu machine using Docker Compose. You’ll monitor CPU, RAM, network, disk, and user logs, and visualize everything in Grafana.

---

## Prerequisites
- Ubuntu (20.04+ recommended)
- Docker & Docker Compose

### Install Docker & Docker Compose
```sh
sudo apt update && sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# Log out and back in, or run: newgrp docker
```

---

## 1. Clone or Prepare the Project Directory

```sh
mkdir ~/monitoring && cd ~/monitoring
# Place docker-compose.yml, prometheus.yml, and promtail-config.yaml here
```

---

## 2. Create Configuration Files

### docker-compose.yml
(Already provided in this repo)

### prometheus.yml
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'promtail'
    static_configs:
      - targets: ['promtail:9080']
```

### promtail-config.yaml
```
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

---

## 3. Start the Monitoring Stack

```sh
docker compose up -d
```

---

## 4. Access the Dashboards

- **Grafana:** http://<your-server-ip>:3000 (default: admin / admin)
- **Prometheus:** http://<your-server-ip>:9090

---

## 5. Configure Grafana Dashboards

### Add Data Sources
1. Go to Grafana → Settings → Data Sources → Add data source
2. Add **Prometheus**:
   - URL: http://prometheus:9090
3. Add **Loki**:
   - URL: http://loki:3100

### Import Dashboards
- Go to Dashboards → Import
- Use dashboard IDs from [Grafana.com](https://grafana.com/grafana/dashboards/):
  - Node Exporter Full: `1860`
  - Loki Syslog: `13639` (or search for "Loki" dashboards)

---

## 6. Prometheus Queries (for Panels)

- **CPU Usage (%):**
  ```
  100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
  ```
- **RAM Usage (bytes):**
  ```
  node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
  ```
- **Network RX/TX (bytes/sec):**
  ```
  rate(node_network_receive_bytes_total[5m])
  rate(node_network_transmit_bytes_total[5m])
  ```
- **Disk Usage (%):**
  ```
  100 * (1 - node_filesystem_avail_bytes / node_filesystem_size_bytes)
  ```

---

## 7. Loki Log Queries (in Grafana → Explore)

- **Show all logs:**
  ```
  {job="varlogs"}
  ```
- **Show sudo usage:**
  ```
  {job="varlogs"} |= "sudo"
  ```
- **Show SSH logins:**
  ```
  {job="varlogs"} |= "sshd"
  ```

---

## 8. Non-Functional Info (System Overview)

- Use Node Exporter Full dashboard for:
  - Hostname, uptime, OS version
  - CPU model, core count
  - Memory, disk, network interfaces

---

## 9. Stopping the Stack

```sh
docker compose down
```

---

## 10. Troubleshooting
- Check logs: `docker compose logs <service>`
- Ensure ports 3000, 9090, 9100, 3100 are open

---

## References
- [Prometheus Docs](https://prometheus.io/docs/)
- [Grafana Docs](https://grafana.com/docs/)
- [Loki Docs](https://grafana.com/docs/loki/latest/)
- [Node Exporter](https://github.com/prometheus/node_exporter)
- [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/)
