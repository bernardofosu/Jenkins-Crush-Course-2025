# 🌐 LDAP & inetOrgPerson Cheat Sheet

---

## 🔐 What is LDAP?

**LDAP (Lightweight Directory Access Protocol)** =  
An open, vendor-neutral, industry-standard protocol used to **access & manage directory information services** over IP networks.

---

## 🔑 Key Points

- ⚡ **Lightweight** → simpler & less resource-heavy than older DAP.  
- 🌳 **Directory** → specialized database for hierarchical data (users, groups, devices).  
- 📡 **Access Protocol** → defines how clients & servers communicate.  

---

## 💻 Common Uses

🔑 Centralized authentication (logins across systems).  
👥 Store org info (users, groups, emails, devices).  
📧 Email / 📞 phone directories.  
📡 VPN authentication.  
🗂️ Backend for **Active Directory**, **OpenLDAP**, **SSO systems**.  

---

## 🕰️ History

📅 1980s → X.500 + DAP (heavy, OSI-based).  
📅 1993 → LDAP introduced → lightweight over TCP/IP.  
⭐ Became the **standard for enterprise identity services**.  

👉 LDAP = **X.500 concepts** + **lighter design**.  

---

## ⚙️ Modern Applications

### 1️⃣ Microsoft Active Directory (AD)
- 🖥️ GUI-based, LDAP + Kerberos.  
- Uses **sAMAccountName** / **userPrincipalName** for login.  

📌 Example:  
