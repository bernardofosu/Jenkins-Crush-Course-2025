# 🌐 Jenkins Public IP Issue & Fix

When the public IP of Jenkins changes, you often get proxy errors 🚨 and Jenkins becomes **slow** until you go into **Manage Jenkins → System → Jenkins URL** and update it manually.

But you can also fix this directly from the server by updating the config file with one command.

---

## 🛠 Update Jenkins URL via command line

**Target file:**  
`/var/lib/jenkins/jenkins.model.JenkinsLocationConfiguration.xml`

Example check:
```bash
ubuntu@ip-172-31-18-129:~$ cat /var/lib/jenkins/jenkins.model.JenkinsLocationConfiguration.xml
<?xml version='1.1' encoding='UTF-8'?>
<jenkins.model.JenkinsLocationConfiguration>
  <adminAddress>ofosubernard848@gmail.com</adminAddress>
  <jenkinsUrl>http://98.89.22.147:8080/</jenkinsUrl>
</jenkins.model.JenkinsLocationConfiguration>
```

Run this command to set Jenkins URL to your **private IP** (or Elastic IP / DNS):

```bash
sudo sed -i 's#<jenkinsUrl>.*</jenkinsUrl>#<jenkinsUrl>http://172.31.18.129:8080/</jenkinsUrl>#' /var/lib/jenkins/jenkins.model.JenkinsLocationConfiguration.xml
```

---

## 🔍 What this does
Finds the line:
```xml
<jenkinsUrl>...</jenkinsUrl>
```
Replaces it entirely with:
```xml
<jenkinsUrl>http://172.31.18.129:8080/</jenkinsUrl>
```

---

## ✅ Apply changes
Restart Jenkins so it loads the new config:
```bash
sudo systemctl restart jenkins
```

---

## ⚡ Pro Tip
Instead of using a changing IP, point a **DNS name** (e.g.,  
`http://jenkins.nakodtech.xyz:8080/`) to your Jenkins instance.  

That way, you **never need to edit the Jenkins URL again** when the IP changes.
