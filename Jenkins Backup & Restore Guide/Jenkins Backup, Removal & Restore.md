# 🚀 Jenkins Backup, Removal & Restore Guide

This guide shows how to **safely back up Jenkins**, completely remove it from a server, and restore it later. This method is useful when:

* 🔄 Upgrading Jenkins
* 🐞 Removing a buggy Jenkins server
* 🖥️ Migrating Jenkins to a new server

✅ The key point: **you won’t lose your jobs and configurations.**

---

## 📦 1. Backup Jenkins

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

➡️ This script:

* Stops Jenkins ⏹️
* Copies all configs & jobs 📂
* Restarts Jenkins ▶️
* Compresses backup into `.tar.gz` 📦

Run:

```bash
chmod +x bkp.sh
./bkp.sh
```

---

## ❌ 2. Remove Jenkins Completely

```bash
# Stop Jenkins service
sudo systemctl stop jenkins

# Remove Jenkins package
sudo apt-get remove --purge jenkins -y

# Remove Jenkins data & configs
sudo rm -rf /var/lib/jenkins
sudo rm -rf /etc/jenkins
sudo rm -rf /var/log/jenkins

# Clean apt sources
sudo rm -f /etc/apt/sources.list.d/jenkins.list
sudo rm -f /etc/apt/keyrings/jenkins-keyring.asc

# Update package list
sudo apt-get update

# (Optional) Remove Jenkins user
sudo deluser --remove-home jenkins

# Final check
which jenkins   # should return no output
```

---

## 🔄 3. Restore Jenkins Backup

### Step 1: Copy Backup

```bash
cd /home/ubuntu/backup
ls
jenkins-2025-04-29.tar.gz
```

### Step 2: Extract Backup

```bash
tar -xvf jenkins-2025-04-29.tar.gz
```

### Step 3: Restore to Jenkins Home

```bash
cp -r jenkins-2025-04-29/* /var/lib/jenkins/
cd /var/lib/jenkins/

# Set ownership\chown -R jenkins:jenkins jenkins/*
```

---

## 🎯 Final Step

Restart Jenkins:

```bash
sudo systemctl start jenkins
```

Go to:

```
http://YOUR_SERVER_IP:8080
```

✅ Jenkins is back with all jobs & configs!

---

## ⚡ Best Practice

While this manual method works, the **best approach** is:

* Push Jenkins home backup to GitHub/remote repo 📤
* Clone or download it on a new server 📥
* Restore jobs & configs quickly 🚀

This makes upgrades, migrations, and disaster recovery much smoother 🔄.

---

📌 **Summary:**

* Backup → Remove → Restore → Resume work without losing jobs.
* Perfect for upgrading or rebuilding Jenkins safely 🛠️.
