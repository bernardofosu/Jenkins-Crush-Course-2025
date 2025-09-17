# 📦 Jenkins Backup & Removal Guide

---

## 🔹 **Backup Script**

📂 File: `backup_jenkins.sh`

```bash
#!/bin/bash

DATE=$(date +%F)
BACKUP_DIR="/home/ubuntu/backup/jenkins-$DATE"

# Stop Jenkins
sudo systemctl stop jenkins

# Backup Jenkins Home
mkdir -p $BACKUP_DIR
cp -r /var/lib/jenkins/* $BACKUP_DIR

# Start Jenkins
sudo systemctl start jenkins

# Optional: Compress backup
tar -czf /home/ubuntu/backup/jenkins-$DATE.tar.gz -C /home/ubuntu/backup jenkins-$DATE
rm -rf $BACKUP_DIR
```

✅ This script:

* Stops Jenkins ⏹️
* Copies `/var/lib/jenkins` → `/home/ubuntu/backup/jenkins-<date>` 📂
* Restarts Jenkins ▶️
* Compresses backup into `.tar.gz` 📦

---

## 🧼 Step-by-Step: Completely Remove Jenkins (Ubuntu/Debian)

### 🔹 1. Stop Jenkins Service

```bash
sudo systemctl stop jenkins
```

### 🔹 2. Remove Jenkins Package

```bash
sudo apt-get remove --purge jenkins -y
```

➡️ `--purge` removes package **and configs**.

---

### 🔹 3. Remove Jenkins Home Directory

```bash
sudo rm -rf /var/lib/jenkins
```

📂 Contains jobs, builds, and runtime data.

---

### 🔹 4. Remove Jenkins Config & Logs

```bash
sudo rm -rf /etc/jenkins
sudo rm -rf /var/log/jenkins
```

* `/etc/jenkins` → configs ⚙️
* `/var/log/jenkins` → logs 📝

---

### 🔹 5. Remove Jenkins Repo & Keys

```bash
sudo rm -f /etc/apt/sources.list.d/jenkins.list
sudo rm -f /etc/apt/keyrings/jenkins-keyring.asc
```

---

### 🔹 6. Update Package List

```bash
sudo apt-get update
```

---

## ✅ Final Check

Run:

```bash
which jenkins
```

➡️ No output = Jenkins completely removed 🧹

---

✨ With this, you can **backup Jenkins safely** before doing a **full clean removal** 🚀
