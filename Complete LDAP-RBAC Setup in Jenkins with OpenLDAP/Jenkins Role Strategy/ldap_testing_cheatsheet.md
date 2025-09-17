# 🔍 LDAP Testing Cheat Sheet (with Emojis)

---

## 1️⃣ Check if a user exists

```bash
ldapsearch -x -H ldap://<host>:389 \
  -D "cn=admin,dc=nakodtech,dc=com" -w 'nakod1234' \
  -b "ou=users,dc=nakodtech,dc=com" "(uid=adminuser)" dn
```

📌 **Sample Output**
```ldif
# adminuser, users, nakodtech.com
dn: uid=adminuser,ou=users,dc=nakodtech,dc=com

# search result
result: 0 Success
numEntries: 1
```
✅ **Meaning**: User `adminuser` exists under `ou=users,dc=nakodtech,dc=com`.

---

## 2️⃣ Test login (bind) as the user

```bash
ldapwhoami -x \
  -D "uid=adminuser,ou=users,dc=nakodtech,dc=com" \
  -w adminpass
```

📌 **Sample Output**
```ldif
dn:uid=adminuser,ou=users,dc=nakodtech,dc=com
```
🔑 **Meaning**: Authentication works → the password is valid.

---

## 3️⃣ Check group membership

```bash
ldapsearch -x -D "cn=admin,dc=nakodtech,dc=com" -w 'nakod1234' \
  -b "ou=groups,dc=nakodtech,dc=com" "(cn=jenkins-admins)" member
```

📌 **Sample Output**
```ldif
# jenkins-admins, groups, nakodtech.com
dn: cn=jenkins-admins,ou=groups,dc=nakodtech,dc=com
member: uid=adminuser,ou=users,dc=nakodtech,dc=com
member: uid=adminuser1,ou=users,dc=nakodtech,dc=com
```

👥 **Meaning**:
- `jenkins-admins` group exists.
- Both `adminuser` and `adminuser1` are members.

---

# ⚡ TL;DR
- 🔎 `ldapsearch ... uid=...` → confirms user exists.
- 🔑 `ldapwhoami ...` → tests login/password works.
- 👥 `ldapsearch ... (cn=group)` → shows which users are in that group.

✨ With this, you’ve got a **step-by-step verification flow**:  
1. Confirm the account exists → 2. Test the password → 3. Confirm group membership.
