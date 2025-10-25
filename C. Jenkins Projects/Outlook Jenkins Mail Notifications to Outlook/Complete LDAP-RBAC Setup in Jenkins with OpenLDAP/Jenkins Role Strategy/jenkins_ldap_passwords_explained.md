# 🔐 Jenkins Authentication with LDAP – Passwords Explained

---

## 1️⃣ Before Enabling LDAP

- 🔑 **Initial Jenkins Admin Password**  
  - Location: `/var/lib/jenkins/secrets/initialAdminPassword`  
  - Purpose: Used **once** to unlock Jenkins after installation and create the first admin user.  
  - ✅ Works only in **Jenkins’ own user database** mode.  

---

## 2️⃣ After Enabling LDAP (OpenLDAP/AD)

- 👤 **Jenkins now delegates authentication to LDAP**  
  - Only LDAP users can log in (e.g., `adminuser1 / adminpass1`, `devuser1 / devpass1`).  
  - The old Jenkins initial password is ❌ no longer valid for login.  

- ⚙️ **LDAP Bind (Manager DN) Password**  
  - Example: `cn=admin,dc=nakodtech,dc=com`  
  - Used internally by Jenkins to query LDAP (search for users/groups).  
  - ❌ Not used for logging into Jenkins.  

- 👥 **LDAP User Accounts**  
  - Example entries:  
    - `uid=adminuser1,ou=users,dc=nakodtech,dc=com` → password = `adminpass1`  
    - `uid=devuser1,ou=users,dc=nakodtech,dc=com` → password = `devpass1`  
    - `uid=viewer1,ou=users,dc=nakodtech,dc=com` → password = `viewerpass1`  
  - ✅ These accounts are how you log in after enabling LDAP.  

---

## 3️⃣ Best Practices

- 🧯 **Keep a Break-Glass Local Admin**  
  - In **Security Realm**, check: *Allow Jenkins own user database* + *LDAP*.  
  - Ensures at least one local admin can log in if LDAP misconfigures.  

- 🔑 **Matrix Authorization**  
  - Add LDAP groups (e.g., `jenkins-admins`, `jenkins-devs`, `jenkins-viewers`).  
  - Always assign at least one known LDAP admin (`adminuser1`) to `jenkins-admins`.  

- ✅ After confirming LDAP works, you can **ignore the old Jenkins initial password**.  

---

## 4️⃣ TL;DR – Password Roles

| 🔑 Password Type                  | 📝 Meaning / Use | ✅ Can Log In? |
|----------------------------------|-----------------|----------------|
| Jenkins Initial Admin Password   | Unlock Jenkins first time | ❌ No (after LDAP) |
| LDAP Bind (Manager DN) Password  | Used internally to query LDAP | ❌ No |
| LDAP User Password (uid=...)     | Real user login credentials | ✅ Yes |

---

✨ With LDAP enabled → **only LDAP users log in**, the Jenkins initial password is **ignored**, and the LDAP bind password is only for background queries.
