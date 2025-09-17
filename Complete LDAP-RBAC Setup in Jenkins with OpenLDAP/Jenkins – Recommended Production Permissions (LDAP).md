# 🔐 Jenkins – Recommended Production Permissions (LDAP)

---

## 👥 Roles & Groups

- 👑 **jenkins-admins** → Full admin (platform owners, SRE)  
- 🧑‍💻 **jenkins-devs** → Build + Read jobs (engineers)  
- 👀 **jenkins-viewers** → Read-only (auditors, managers)  
- 🧯 **Break-glass user:** `adminuser` (explicit Administer)  

---

## ✅ Matrix-Based Security (minimal, safe)

| User/Group         | Overall → Administer | Overall → Read | Job → Read | Job → Build | Credentials → View |
|--------------------|----------------------|----------------|------------|-------------|---------------------|
| @ jenkins-admins   | ✅                   | ✅             | ✅         | ✅          | ✅                  |
| @ jenkins-devs     |                      | ✅             | ✅         | ✅ (optional) | ✅                  |
| @ jenkins-viewers  |                      | ✅             | ✅         |             |                     |
| adminuser (user)   | ✅                   | ✅             | ✅         | ✅          | ✅                  |
| Authenticated      |                      |                |            |             |                     |
| Anonymous          |                      |                |            |             |                     |

⚡ **Tip:** In the UI → *Add group…* (no `@` needed).  
Add `adminuser` via *Add user…*.  
🚫 Keep **Anonymous** empty in production.  

---

## 🧰 Suggested Extras (optional but nice)

- 🧪 **Run → Replay** → only for admins  
- 🧱 **Agent configure/provision** → only admins  
- 🧾 **View → Read/Create** → devs if they manage team views  
- 🧰 **SCM → Tag** → restrict to admins/release engineers  

---

## 🧯 Break-Glass Rationale

- Keep one **explicit Admin user** (`adminuser`) even if they’re in `jenkins-admins`.  
- If LDAP group lookup breaks → guaranteed access.  
- 🔄 Review quarterly & rotate that user’s password.  

---

## 🧪 LDAP Sanity Checks

🔎 Is the user resolvable?  
```bash
ldapsearch -x -H ldap://<host>:389 -D "cn=admin,dc=nakodtech,dc=com" -w '<bind-pass>' \
  -b "ou=users,dc=nakodtech,dc=com" "(uid=adminuser)" dn
