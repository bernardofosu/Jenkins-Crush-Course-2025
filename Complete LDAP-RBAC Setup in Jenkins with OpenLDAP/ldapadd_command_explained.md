# 🔐 `ldapadd` Command Explained with Emojis

Command:

```bash
ldapadd -x -D "cn=admin,dc=nakodtech,dc=com" -W -f base.ldif
```

---

## 🧩 Parts of the Command

| ⚙️ Part | 📝 Full Meaning | 📌 Explanation |
|---------|----------------|----------------|
| **ldapadd** | LDAP Add Utility | Tool to **add entries** into LDAP from an `.ldif` file. |
| **-x** | Simple Authentication | Uses simple **username + password** auth (not Kerberos/SASL). |
| **-D "cn=admin,dc=nakodtech,dc=com"** | Bind DN | The **Distinguished Name** of the user you log in as. Here: LDAP **admin account**. |
| **-W** | Prompt for Password | Securely prompts for the password of the DN. You type it in, it’s not shown. |
| **-f base.ldif** | Input File | The `.ldif` file with entries (users, groups, base tree) that will be added. |

---

## ⚡ What Happens Step by Step

1. 🏃 Run `ldapadd` → start LDAP add process.  
2. 🔑 Log in as **admin** (`cn=admin,dc=nakodtech,dc=com`).  
3. 🔒 Prompt for admin **password** (`-W`).  
4. 📂 Read entries from **`base.ldif`** file.  
5. 🌳 Insert those entries into the LDAP Directory Information Tree (DIT).  

---

## 🌳 Resulting LDAP Tree

```
dc=nakodtech,dc=com   🌐 (root domain)
 ├── ou=users        👤 container for users
 └── ou=groups       👥 container for groups
```

---

✅ In short: **This command logs in as admin and adds the entries from `base.ldif` into your LDAP server.**
