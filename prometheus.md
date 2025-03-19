I get you! Let's fix this properly. You want **one complete Markdown file** of your setup guide (Prometheus, Grafana, Node Exporter on AWS EC2), formatted cleanly in `.md`, right?

I'll give you the **whole thing in one go**, no splitting. Here's the **complete Markdown version**:

---

```markdown
# 🖥️ Architecture Overview

- **Node Exporter**: Collects system metrics.
- **Prometheus**: Scrapes metrics from Node Exporter.
- **Grafana**: Visualizes Prometheus data.

---

## ⚙️ Prerequisites

✅ 1 EC2 instance (Ubuntu 20.04/22.04)

✅ Open these ports in **EC2 Security Group**:
- `9090` → Prometheus  
- `9100` → Node Exporter  
- `3000` → Grafana  

---

## 🔧 Step 1: Update Packages

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🔧 Step 2: Install Prometheus

### 2.1 Create a user for Prometheus:
```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

### 2.2 Create directories:
```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

### 2.3 Download Prometheus:
```bash
cd /tmp
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4
```
Pick the latest version URL and download:
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
```

### 2.4 Extract and move files:
```bash
tar -xvf prometheus-*.tar.gz
cd prometheus-*.linux-amd64
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp prometheus.yml /etc/prometheus
```

### 2.5 Change ownership:
```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

### 2.6 Create Prometheus Service:
```bash
sudo nano /etc/systemd/system/prometheus.service
```

Paste the following content:
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

### 2.7 Start and enable Prometheus:
```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

✅ Verify Prometheus:  
`http://<EC2_PUBLIC_IP>:9090`

---

## 🔧 Step 3: Install Node Exporter

### 3.1 Create a user:
```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

### 3.2 Download Node Exporter:
```bash
cd /tmp
curl -s https://api.github.com/repos/prometheus/node_exporter/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4
```
Pick the latest version URL and download:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz
```

### 3.3 Extract and move files:
```bash
tar -xvf node_exporter-*.tar.gz
cd node_exporter-*.linux-amd64
sudo cp node_exporter /usr/local/bin/
```

### 3.4 Change ownership:
```bash
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### 3.5 Create Node Exporter service:
```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste the following content:
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
```

### 3.6 Start and enable Node Exporter:
```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

✅ Verify Node Exporter:  
`http://<EC2_PUBLIC_IP>:9100/metrics`

---

## 🔧 Step 4: Configure Prometheus to scrape Node Exporter

### 4.1 Edit Prometheus config:
```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add the following under `scrape_configs`:
```yaml
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

💡 If Node Exporter is on another server, use its private IP instead.

### 4.2 Restart Prometheus:
```bash
sudo systemctl restart prometheus
```

---

## 🔧 Step 5: Install Grafana

### 5.1 Add Grafana APT repo:
```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
```

### 5.2 Install Grafana:
```bash
sudo apt-get install grafana -y
```

### 5.3 Start and enable Grafana:
```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

✅ Access Grafana:  
`http://<EC2_PUBLIC_IP>:3000`  
Login: `admin / admin`  
(Change the password after first login)

---

## 🔧 Step 6: Connect Prometheus to Grafana

1. Go to **Grafana → Settings → Data Sources → Add data source**
2. Select **Prometheus**
3. URL → `http://localhost:9090`  
   (or your EC2 IP if remote)
4. Click **Save & Test**

---

## 🔧 Step 7: Import Dashboards in Grafana

1. Go to **Create → Import**
2. Use dashboard ID → `1860`  
   (Node Exporter Full)
3. Select **Prometheus** as the data source
4. 🎉 Get system metrics (CPU, memory, disk, etc.)

---

## ✅ Done! You have a working Monitoring Stack! ✅

---

## 📝 Summary Table

| Component      | Port  | Purpose                   |
|----------------|-------|---------------------------|
| Prometheus     | 9090  | Scrapes & stores metrics  |
| Node Exporter  | 9100  | Exposes Linux metrics     |
| Grafana        | 3000  | Visualizes metrics        |

---
``
