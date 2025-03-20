email notification

Sure! Below is the **Markdown (.md)** format you can directly use in your GitHub repo.

---

```markdown
# 🚨 Alertmanager with Prometheus: Step-by-Step Guide

This guide helps you set up **Alertmanager** with **Prometheus** and configure **Slack and Email notifications**.

---

## ✅ What You'll Do
- Install and configure Alertmanager
- Connect Prometheus to Alertmanager
- Create Prometheus alert rules
- Send notifications to Slack and Email
- Test and verify alerts

---

## 🟢 Step 1: Install Alertmanager

### 1.1 Download and Extract Alertmanager
```bash
cd /tmp
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar -xvf alertmanager-0.27.0.linux-amd64.tar.gz
sudo mv alertmanager-0.27.0.linux-amd64 /opt/alertmanager
```

### 1.2 Create User and Directories
```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin alertmanager
sudo mkdir /etc/alertmanager /var/lib/alertmanager
sudo chown -R alertmanager:alertmanager /etc/alertmanager /var/lib/alertmanager
```

### 1.3 Symlink Binaries
```bash
sudo ln -s /opt/alertmanager/alertmanager /usr/local/bin/alertmanager
sudo ln -s /opt/alertmanager/amtool /usr/local/bin/amtool
```

---

## 🟢 Step 2: Configure Alertmanager

### 2.1 Create Alertmanager Configuration File
```bash
sudo nano /etc/alertmanager/alertmanager.yml
```

### Example Configuration for Slack and Email:
```yaml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'your-email@gmail.com'
  smtp_auth_username: 'your-email@gmail.com'
  smtp_auth_password: 'your-app-password'

route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 1m
  repeat_interval: 1h
  receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        channel: '#alerts'
  - name: 'email-notifications'
    email_configs:
      - to: 'receiver-email@gmail.com'
```

> 🔒 **Important**  
> - `smtp_auth_password` must be a **Gmail App Password**  
> - Replace `api_url` with your actual Slack webhook URL

---

## 🟢 Step 3: Create systemd Service for Alertmanager

```bash
sudo nano /etc/systemd/system/alertmanager.service
```

Paste this:
```ini
[Unit]
Description=Alertmanager
After=network.target

[Service]
User=alertmanager
Group=alertmanager
WorkingDirectory=/var/lib/alertmanager
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager

Restart=always

[Install]
WantedBy=multi-user.target
```

---

## 🟢 Step 4: Start and Enable Alertmanager
```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

Check status:
```bash
sudo systemctl status alertmanager
```

Access Alertmanager UI at:  
```
http://<server-ip>:9093
```

---

## 🟢 Step 5: Configure Prometheus to Use Alertmanager

### 5.1 Edit Prometheus Config
```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add or edit the following block:
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

---

## 🟢 Step 6: Create Prometheus Alert Rules (High CPU Example)

### 6.1 Create Rules File
```bash
sudo nano /etc/prometheus/alert.rules.yml
```

Paste this rule:
```yaml
groups:
  - name: CPU_Usage
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100) > 70
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is above 70% for more than 2 minutes."
```

### 6.2 Link the Rule File in Prometheus Config
Edit `prometheus.yml`:
```yaml
rule_files:
  - /etc/prometheus/alert.rules.yml
```

---

## 🟢 Step 7: Restart Prometheus
```bash
sudo systemctl restart prometheus
```

Check alerts page:  
```
http://<server-ip>:9090/alerts
```

---

## 🟢 Step 8: Test Alertmanager Notifications

### 8.1 Manually Trigger a Test Alert
```bash
amtool --alertmanager.url=http://localhost:9093 alert add TestAlert instance=localhost summary="Test Alert"
```

### 8.2 Stress Test to Simulate High CPU Usage
```bash
sudo apt install stress -y
stress --cpu 2 --timeout 300
```

---

## 🟢 Step 9: Verify Everything Works
- Prometheus Alerts: `http://<server-ip>:9090/alerts`
- Alertmanager UI: `http://<server-ip>:9093`
- Slack Channel: `#alerts`
- Email Inbox: `receiver-email@gmail.com`

---

## 🟢 Step 10: Clean Up Old Alertmanager Data (Optional)
```bash
sudo systemctl stop alertmanager
sudo rm -rf /var/lib/alertmanager/*
sudo systemctl start alertmanager
```

---

## ✅ Additional Notes
- Gmail SMTP requires an **App Password**, not your Google account password.
- Slack Webhooks need to be configured in your Slack workspace.
- Ensure Prometheus is scraping node exporters for system metrics.

---

## ✅ Next Steps
- Set up **Grafana Alerts**
- Configure **inhibit_rules** in Alertmanager
- Integrate **PagerDuty**, **Telegram**, **Opsgenie**, etc.

---

## 📚 References
- [Prometheus Docs](https://prometheus.io/docs/)
- [Alertmanager Docs](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Slack Incoming Webhooks](https://api.slack.com/messaging/webhooks)
- [Gmail App Passwords](https://support.google.com/accounts/answer/185833?hl=en)

---
```

---

✅ Save the above as `Alertmanager-Prometheus-Setup.md` and commit to your GitHub repo.  
If you need a **Telegram** or **Grafana** alert setup next, let me know!
