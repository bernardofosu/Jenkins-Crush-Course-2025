# 🛠️ Infra & Monitoring Tools Installation Guide

Here’s a **Canva-style breakdown with emojis** for setting up monitoring, DevOps, and database tools.

---

## 1️⃣ Prometheus 📈

```bash
cd /tmp
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.55.1/prometheus-2.55.1.linux-amd64.tar.gz
tar -xzf prometheus-*.tar.gz
sudo mv prometheus-*/prometheus /usr/local/bin/
sudo mv prometheus-*/promtool /usr/local/bin/
prometheus --version
```

🔍 Open-source monitoring & alerting toolkit.

---

## 2️⃣ Grafana 📊

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo wget -q -O /etc/apt/trusted.gpg.d/grafana.gpg https://packages.grafana.com/gpg.key
sudo apt-get update -y
sudo apt-get install -y grafana
grafana-server -v
```

🎨 Analytics & visualization platform.

---

## 3️⃣ Splunk 🔎

```bash
cd /tmp
wget -O splunk.tgz 'https://download.splunk.com/products/splunk/releases/9.2.1/linux/splunk-9.2.1-ae6821b7c3f3-Linux-x86_64.tgz'
sudo tar -xzf splunk.tgz -C /opt
/opt/splunk/bin/splunk version
```

📡 Enterprise log management & SIEM.

---

## 4️⃣ Docker 🐳 (Official Repo)

```bash
sudo apt-get remove -y docker docker-engine docker.io containerd runc || true
sudo apt-get install -y ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
docker --version
```

📦 Containers runtime from the official repo.

---

## 5️⃣ Kubernetes (kubectl) ☸️

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

⚙️ CLI for managing Kubernetes clusters.

---

## 6️⃣ Helm ⛵

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

📦 Kubernetes package manager.

---

## 7️⃣ Terraform 🌍

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt-get update -y
sudo apt-get install -y terraform
terraform -version
```

🏗️ Infrastructure as Code (IaC) tool.

---

## 8️⃣ Ansible 🤖

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt-get install -y ansible
ansible --version
```

📡 Automation & configuration management.

---

## 9️⃣ Node.js 🟩

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
```

⚡ JavaScript runtime for modern apps.

---

## 🔟 PostgreSQL 🐘

```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update -y
sudo apt-get install -y postgresql-15
psql --version
```

📂 Advanced open-source relational database.

---

## 1️⃣1️⃣ MySQL 🐬

```bash
wget https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb -O /tmp/mysql-apt-config.deb
sudo DEBIAN_FRONTEND=noninteractive dpkg -i /tmp/mysql-apt-config.deb
sudo apt-get update -y
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y mysql-server
sudo systemctl enable mysql
sudo systemctl start mysql
mysql --version
```

🗄️ Popular relational database (official repo).

---

## 1️⃣2️⃣ SQLite 🗂️

```bash
sudo apt-get update -y
sudo apt-get install -y sqlite3
sqlite3 --version
```

📦 Lightweight, file-based SQL database.

---

# ⚡ Summary

✅ **Monitoring & Logging:** Prometheus, Grafana, Splunk
✅ **Containers & Orchestration:** Docker, Kubernetes (kubectl), Helm
✅ **Infrastructure & Automation:** Terraform, Ansible
✅ **App Runtime:** Node.js
✅ **Databases:** PostgreSQL, MySQL, SQLite

🔧 With these 12 tools, your server is equipped for **full-stack DevOps, monitoring, automation, and databases**!
