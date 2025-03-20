7. Set Up Alerts in Prometheus
a) Add Rule to Prometheus Alert Rules
Create file /etc/prometheus/alert.rules.yml

groups:
- name: resource-alerts
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is above 80% for more than 2 minutes."

  - alert: HighMemoryUsage
    expr: node_memory_Active_bytes / node_memory_MemTotal_bytes * 100 > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High Memory usage on {{ $labels.instance }}"
      description: "Memory usage is above 80% for more than 2 minutes."
