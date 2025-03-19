

# 🚀 Prometheus + Grafana + Node Exporter on AWS with Terraform

Automate the monitoring stack deployment (**Prometheus**, **Grafana**, **Node Exporter**) on an **AWS EC2 instance** using **Terraform** and **User Data** scripts.

---

## 📋 What We'll Do

✅ Launch an **Ubuntu EC2 instance**  
✅ Open required ports via **Security Group**  
✅ Install and configure **Prometheus**, **Grafana**, and **Node Exporter** via **User Data**  
✅ Output **Public IP** and service URLs for quick access  

---

## 🗂️ Folder Structure

```
prometheus-grafana-ec2/
├── main.tf
├── variables.tf
├── outputs.tf
└── user-data.sh
```

---

## 🔧 Terraform Configuration

### `variables.tf`

```hcl
variable "aws_region" {
  default = "ap-south-1"
}

variable "instance_type" {
  default = "t2.micro"
}

variable "key_name" {
  description = "AWS Key Pair name for SSH access"
}
```

---

### `main.tf`

```hcl
provider "aws" {
  region = var.aws_region
}

resource "aws_security_group" "monitoring_sg" {
  name        = "prometheus-grafana-sg"
  description = "Allow Prometheus, Grafana, Node Exporter, and SSH"

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Prometheus"
    from_port   = 9090
    to_port     = 9090
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Node Exporter"
    from_port   = 9100
    to_port     = 9100
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Grafana"
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "monitoring" {
  ami                         = "ami-0f58b397bc5c1f2e8" # Ubuntu 22.04 (ap-south-1)
  instance_type               = var.instance_type
  key_name                    = var.key_name
  associate_public_ip_address = true

  vpc_security_group_ids = [aws_security_group.monitoring_sg.id]

  user_data = file("user-data.sh")

  tags = {
    Name = "Prometheus-Grafana-NodeExporter"
  }
}
```

---

### `outputs.tf`

```hcl
output "instance_public_ip" {
  description = "Public IP of EC2 instance"
  value       = aws_instance.monitoring.public_ip
}

output "grafana_url" {
  value = "http://${aws_instance.monitoring.public_ip}:3000"
}

output "prometheus_url" {
  value = "http://${aws_instance.monitoring.public_ip}:9090"
}

output "node_exporter_url" {
  value = "http://${aws_instance.monitoring.public_ip}:9100"
}
```

---

## ⚙️ `user-data.sh` (Bootstrap Script)

```bash
#!/bin/bash
# Update and install basic tools
apt-get update -y && apt-get upgrade -y
apt-get install -y wget curl unzip gnupg software-properties-common

# Install Prometheus
useradd --no-create-home --shell /bin/false prometheus
mkdir /etc/prometheus /var/lib/prometheus
cd /tmp
PROM_VERSION="2.52.0"
wget https://github.com/prometheus/prometheus/releases/download/v${PROM_VERSION}/prometheus-${PROM_VERSION}.linux-amd64.tar.gz
tar -xzf prometheus-${PROM_VERSION}.linux-amd64.tar.gz
cd prometheus-${PROM_VERSION}.linux-amd64
cp prometheus promtool /usr/local/bin/
cp -r consoles console_libraries prometheus.yml /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus /usr/local/bin/prometheus /usr/local/bin/promtool

cat <<EOF > /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus/

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now prometheus

# Install Node Exporter
useradd --no-create-home --shell /bin/false node_exporter
cd /tmp
NODE_EXPORTER_VERSION="1.8.0"
wget https://github.com/prometheus/node_exporter/releases/download/v${NODE_EXPORTER_VERSION}/node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz
tar -xzf node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz
cd node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64
cp node_exporter /usr/local/bin/
chown node_exporter:node_exporter /usr/local/bin/node_exporter

cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now node_exporter

# Install Grafana
apt-get install -y apt-transport-https
wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
apt-get update -y
apt-get install -y grafana
systemctl enable --now grafana-server
```

---

## 🏃‍♂️ Terraform Commands

1. **Initialize Terraform**  
   ```bash
   terraform init
   ```

2. **Validate Configuration**  
   ```bash
   terraform validate
   ```

3. **Plan the Deployment**  
   ```bash
   terraform plan -var="key_name=YOUR_KEY_PAIR_NAME"
   ```

4. **Apply the Deployment**  
   ```bash
   terraform apply -var="key_name=YOUR_KEY_PAIR_NAME"
   ```

---

## 🔗 Access Your Services

| Service      | URL                                     |
| ------------ | --------------------------------------- |
| Grafana      | `http://<EC2_PUBLIC_IP>:3000` (admin/admin) |
| Prometheus   | `http://<EC2_PUBLIC_IP>:9090`          |
| Node Exporter| `http://<EC2_PUBLIC_IP>:9100/metrics`  |

---

## 🎉 What's Next?

- 🔔 Add **Alertmanager** & integrate with **Slack / Email**  
- 📈 Preconfigure **Grafana Dashboards**  
- 📦 Add **EFS** or **S3** for storage persistence  
- 🌐 Setup **Multi-AZ deployments** for high availability  

---
