# 🛠️ Jenkins ↔️ LDAP on Private IP (EC2) — Fix Login Failures

When Jenkins and your LDAP server run on the **same EC2**, point Jenkins to the **private IP** (e.g., `172.31.18.129:389`) so logins keep working even if the **public IP** changes.

---

## 🚨 Symptom — Login fails after public IP change

You try to log in to Jenkins with your LDAP user and it fails. In **`/var/log/jenkins/jenkins.log`** you’ll see errors like:

```text
SEVERE  hudson.security.LDAPSecurityRealm$LDAPUserDetailsService
org.springframework.security.authentication.AuthenticationServiceException: Could not retrieve user details; nested exception is
javax.naming.CommunicationException: ec2-3-90-77-10.compute-1.amazonaws.com:389
[Root exception is java.net.UnknownHostException: ec2-3-90-77-10.compute-1.amazonaws.com]
    at hudson.security.LDAPSecurityRealm$LDAPUserDetailsService.loadUserByUsername(LDAPSecurityRealm.java:...)
Caused by: javax.naming.CommunicationException: ec2-3-90-77-10.compute-1.amazonaws.com:389
Root exception is java.net.ConnectException: Connection timed out (Connection timed out)
```

Or, if the public IP changed and DNS hasn’t updated:

```text
javax.naming.CommunicationException: 3.90.77.10:389
Root exception is java.net.ConnectException: Connection refused
```

**Why?** Jenkins was configured to use the **public IP / hostname** of LDAP. When that changes, Jenkins can’t reach LDAP → logins fail.

---

## ✅ Solution — Use the EC2 **private IP** for LDAP

### 1) Stop Jenkins and back up your config
```bash
sudo systemctl stop jenkins
sudo cp /var/lib/jenkins/config.xml /var/lib/jenkins/config.xml.bak.$(date +%F-%H%M)
```

### 2) Replace the `<server>` line with the private IP
```bash
sudo sed -i 's#<server>ldap://[^<]\+</server>#<server>ldap://172.31.18.129:389</server>#' /var/lib/jenkins/config.xml
```

### 3) Trim trailing spaces in `<rootDN>` (if any)
```bash
sudo sed -i 's#<rootDN>dc=nakodtech,dc=com[[:space:]]*</rootDN>#<rootDN>dc=nakodtech,dc=com</rootDN>#' /var/lib/jenkins/config.xml
```

> 📝 Make sure your **Bind DN** and **User search base** are correct (e.g., `cn=admin,dc=nakodtech,dc=com` and `ou=users,dc=nakodtech,dc=com`).

### 4) Start Jenkins
```bash
sudo systemctl start jenkins
```

---

## 🔍 Preflight checks (LDAP on private IP)

Before testing Jenkins login, verify LDAP directly from the Jenkins host:

```bash
ldapsearch -x -H ldap://172.31.18.129:389   -D "cn=admin,dc=nakodtech,dc=com" -W   -b "dc=nakodtech,dc=com" "(uid=adminuser)"
```

Optional quick port test:
```bash
nc -zv 172.31.18.129 389
# or
telnet 172.31.18.129 389
```

If these fail, check:
- `sudo systemctl status slapd` (OpenLDAP)
- **Security Groups**: allow TCP **389** from the Jenkins instance’s **security group** (intra-VPC).
- Local firewall (UFW/iptables) and SELinux policies.

---

## 👤 Then try Jenkins login
Open Jenkins UI and log in with your LDAP user:

- **Username:** `adminuser`  
- **Password:** *(the one you set/reset via `ldappasswd`)*

If it still fails, re-check `/var/lib/jenkins/config.xml` for the `<server>` value and run the **Preflight checks** again.

---

## 🧯 Rollback (if needed)
Restore your backup and restart:
```bash
sudo systemctl stop jenkins
sudo cp /var/lib/jenkins/config.xml.bak.* /var/lib/jenkins/config.xml
sudo systemctl start jenkins
```

---

## 💡 Best practices
- Use **private IPs** or **VPC-internal DNS** for intra-VPC services.
- Avoid hardcoding **public IPs** in service configs.
- Keep a **break-glass local admin** in Jenkins for emergencies.
- Document your **Bind DN**, **search bases**, and **group filters**.
