https://chatgpt.com/canvas/shared/67db2a76fa8c819183530bd16de1ec9d

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Monitoring Stack Setup (Prometheus, Node Exporter, Alertmanager, Grafana)</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
      color: #333;
    }
    header {
      background-color: #007acc;
      color: #fff;
      padding: 20px 0;
      text-align: center;
    }
    .container {
      padding: 20px;
      max-width: 900px;
      margin: auto;
      background-color: #fff;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    h1, h2, h3 {
      color: #007acc;
    }
    pre {
      background-color: #333;
      color: #fff;
      padding: 10px;
      overflow-x: auto;
      border-radius: 4px;
    }
    code {
      color: #f8f8f2;
    }
    a {
      color: #007acc;
    }
    ul {
      margin-left: 20px;
    }
    footer {
      text-align: center;
      padding: 10px;
      font-size: 14px;
      color: #888;
    }
  </style>
</head>
<body>

<header>
  <h1>Monitoring Stack Setup</h1>
  <p>Prometheus | Node Exporter | Alertmanager | Grafana</p>
</header>

<div class="container">
  <h2>🛠️ Prerequisites</h2>
  <ul>
    <li>AWS EC2 instance (Ubuntu 20.04 or later)</li>
    <li>Open security groups:
      <ul>
        <li>9090 (Prometheus)</li>
        <li>9100 (Node Exporter)</li>
        <li>9093 (Alertmanager)</li>
        <li>3000 (Grafana)</li>
      </ul>
    </li>
    <li>SSH access and internet connectivity</li>
  </ul>

  <h2>1️⃣ Prometheus Installation</h2>
  <h3>Step 1: Update and create Prometheus user</h3>
  <pre><code>sudo apt update -y
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus</code></pre>

  <h3>Step 2: Download and extract Prometheus</h3>
  <pre><code>cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar -xvf prometheus-2.52.0.linux-amd64.tar.gz
cd prometheus-2.52.0.linux-amd64</code></pre>

  <h3>Step 3: Move binaries and config files</h3>
  <pre><code>sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp prometheus.yml /etc/prometheus</code></pre>

  <h3>Step 4: Set permissions</h3>
  <pre><code>sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool</code></pre>

  <h3>Step 5: Create Prometheus systemd service</h3>
  <pre><code>sudo nano /etc/systemd/system/prometheus.service</code></pre>

  <p>Paste the following content:</p>
  <pre><code>[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target</code></pre>

  <h3>Step 6: Start and enable Prometheus</h3>
  <pre><code>sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus</code></pre>

  <p>Access Prometheus UI: <strong>http://&lt;EC2_PUBLIC_IP&gt;:9090</strong></p>

  <h2>2️⃣ Node Exporter Installation</h2>
  <h3>Step 1: Create user</h3>
  <pre><code>sudo useradd --no-create-home --shell /bin/false node_exporter</code></pre>

  <h3>Step 2: Download Node Exporter</h3>
  <pre><code>cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz
tar -xvf node_exporter-1.8.0.linux-amd64.tar.gz
cd node_exporter-1.8.0.linux-amd64</code></pre>

  <h3>Step 3: Move binary and set permissions</h3>
  <pre><code>sudo cp node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter</code></pre>

  <h3>Step 4: Create systemd service</h3>
  <pre><code>sudo nano /etc/systemd/system/node_exporter.service</code></pre>

  <p>Paste the following content:</p>
  <pre><code>[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target</code></pre>

  <h3>Step 5: Start and enable Node Exporter</h3>
  <pre><code>sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter</code></pre>

  <p>Access Node Exporter: <strong>http://&lt;EC2_PUBLIC_IP&gt;:9100</strong></p>

  <h2>3️⃣ Alertmanager Installation</h2>
  <h3>Step 1: Create user and directories</h3>
  <pre><code>sudo useradd --no-create-home --shell /bin/false alertmanager
sudo mkdir /etc/alertmanager /var/lib/alertmanager</code></pre>

  <h3>Step 2: Download and extract Alertmanager</h3>
  <pre><code>cd /tmp
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar -xvf alertmanager-0.27.0.linux-amd64.tar.gz
cd alertmanager-0.27.0.linux-amd64</code></pre>

  <h3>Step 3: Move files and set permissions</h3>
  <pre><code>sudo cp alertmanager amtool /usr/local/bin/
sudo cp alertmanager.yml /etc/alertmanager/
sudo chown -R alertmanager:alertmanager /etc/alertmanager /var/lib/alertmanager
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager /usr/local/bin/amtool</code></pre>

  <h3>Step 4: Create systemd service</h3>
  <pre><code>sudo nano /etc/systemd/system/alertmanager.service</code></pre>

  <p>Paste the following content:</p>
  <pre><code>[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager/

[Install]
WantedBy=multi-user.target</code></pre>

  <h3>Step 5: Start and enable Alertmanager</h3>
  <pre><code>sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager</code></pre>

  <p>Access Alertmanager: <strong>http://&lt;EC2_PUBLIC_IP&gt;:9093</strong></p>

  <h2>4️⃣ Grafana Installation</h2>
  <h3>Step 1: Install dependencies and add repository</h3>
  <pre><code>sudo apt-get install -y software-properties-common apt-transport-https wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt-get update</code></pre>

  <h3>Step 2: Install Grafana</h3>
  <pre><code>sudo apt-get install grafana -y</code></pre>

  <h3>Step 3: Start and enable Grafana</h3>
  <pre><code>sudo systemctl start grafana-server
sudo systemctl enable grafana-server</code></pre>

  <p>Access Grafana UI: <strong>http://&lt;EC2_PUBLIC_IP&gt;:3000</strong></p>
  <ul>
    <li>Default username: <strong>admin</strong></li>
    <li>Default password: <strong>admin</strong></li>
  </ul>

  <h2>✅ Final Notes</h2>
  <ul>
    <li>Configure Prometheus targets in <code>/etc/prometheus/prometheus.yml</code></li>
    <li>Configure Alertmanager rules in <code>/etc/alertmanager/alertmanager.yml</code></li>
    <li>Open necessary ports in AWS Security Groups</li>
    <li>Create Grafana dashboards and connect Prometheus as a data source</li>
  </ul>

  <h2>🔗 Useful Links</h2>
  <ul>
    <li><a href="https://prometheus.io/" target="_blank">Prometheus</a></li>
    <li><a href="https://github.com/prometheus/node_exporter" target="_blank">Node Exporter</a></li>
    <li><a href="https://prometheus.io/docs/alerting/latest/alertmanager/" target="_blank">Alertmanager</a></li>
    <li><a href="https://grafana.com/grafana/download" target="_blank">Grafana</a></li>
  </ul>

</div>

<footer>
  <p>© 2025 Monitoring Guide | Made by Ritik 🚀</p>
</footer>

</body>
</html>
