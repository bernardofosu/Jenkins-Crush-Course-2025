# 🛠️ Jenkins Backup & Restore Guide

---

## 🔹 **1. Backup Options**

* 📂 **Entire Jenkins Home** → `/var/lib/jenkins/` (configs, jobs, plugins, secrets)
* 📁 **Single Job** → `/var/lib/jenkins/jobs/<job-name>`
* 📑 **Multiple Jobs** → copy specific subfolders under `jobs/`

---

## 🔹 **2. Backup Steps**

### ✅ Copy Jenkins data to safe location

```bash
# Copy entire Jenkins home
sudo cp -r /var/lib/jenkins /home/ubuntu/jenkins/backup_all

# Copy one job
sudo cp -r /var/lib/jenkins/jobs/MyJob /home/ubuntu/jenkins/job_backup/

# Copy multiple jobs
sudo cp -r /var/lib/jenkins/jobs/{Job1,Job2,Job3} /home/ubuntu/jenkins/job_backup/
```

### ✅ Push Backup to Git Repo

```bash
cd /home/ubuntu/jenkins/
git init
git remote add origin https://github.com/youruser/jenkins-backup.git
git add .
git commit -m "Daily Jenkins backup"
git push origin main
```

💡 Use `.gitignore` to skip `workspace/` & `builds/` to reduce repo size.

---

## 🔹 **3. Restore on New Jenkins Server**

### ✅ Clone Repo

```bash
cd /home/ubuntu/
git clone https://github.com/youruser/jenkins-backup.git jenkins-restore
```

### ✅ Adjust Ownership

```bash
sudo chown -R jenkins:jenkins jenkins-restore
```

### ✅ Stop Jenkins Before Replace

```bash
sudo systemctl stop jenkins
```

### ✅ Copy Back to Jenkins Home

```bash
# Restore entire Jenkins folder
sudo cp -r /home/ubuntu/jenkins-restore/* /var/lib/jenkins/

# Restore one job
sudo cp -r /home/ubuntu/jenkins-restore/jobs/MyJob /var/lib/jenkins/jobs/

# Restore multiple jobs
sudo cp -r /home/ubuntu/jenkins-restore/jobs/{Job1,Job2} /var/lib/jenkins/jobs/
```

### ✅ Start Jenkins Again

```bash
sudo systemctl start jenkins
```

---

## 🔹 **4. Daily Task Strategy**

* 🕒 Automate backup with **cron job** (daily at midnight):

  ```bash
  0 0 * * * /usr/bin/rsync -a --exclude 'workspace/*' --exclude 'builds/*' /var/lib/jenkins/ /home/ubuntu/jenkins/daily_backup/
  ```
* 📦 Keep repo small by backing up **configs + jobs** only.
* 🔐 Always check `ownership → jenkins:jenkins`.

---

✨ **Summary:**

* Full backup = `/var/lib/jenkins/`
* Small backup = selected `jobs/` + configs
* Restore = stop Jenkins → copy → chown → start Jenkins 🚀
