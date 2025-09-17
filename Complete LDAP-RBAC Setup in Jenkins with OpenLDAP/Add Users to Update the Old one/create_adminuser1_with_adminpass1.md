# 👤 Create `adminuser1` & Add to `jenkins-admins` (LDAP + Jenkins)

---

## 1️⃣ Generate Password Hash  
🔑 Hash password `adminpass1`  

```bash
slappasswd -h {SSHA} -s adminpass1
```  

👉 Copy the full output (e.g., `{SSHA}abc...=`)  

---

## 2️⃣ Create the User  
📂 **adminuser1.ldif**  

```ldif
dn: uid=adminuser1,ou=users,dc=nakodtech,dc=com
objectClass: inetOrgPerson
uid: adminuser1
sn: Admin1
cn: Admin1 User
userPassword: {SSHA}PASTE_HASH_FROM_SLAPPASSWD_HERE
```  

🚀 Add user:  
```bash
ldapadd -x -D "cn=admin,dc=nakodtech,dc=com" -W -f adminuser1.ldif
```  

⚡ If user exists → reset password:  

📂 **reset-adminuser1-pass.ldif**  
```ldif
dn: uid=adminuser1,ou=users,dc=nakodtech,dc=com
changetype: modify
replace: userPassword
userPassword: {SSHA}PASTE_HASH_FROM_SLAPPASSWD_HERE
```  

```bash
ldapmodify -x -D "cn=admin,dc=nakodtech,dc=com" -W -f reset-adminuser1-pass.ldif
```  

---

## 3️⃣ Add User to Group  
📂 **add-adminuser1-to-admins.ldif**  

```ldif
dn: cn=jenkins-admins,ou=groups,dc=nakodtech,dc=com
changetype: modify
add: member
member: uid=adminuser1,ou=users,dc=nakodtech,dc=com
```  

🚀 Apply:  
```bash
ldapmodify -x -D "cn=admin,dc=nakodtech,dc=com" -W -f add-adminuser1-to-admins.ldif
```  

⚠️ Errors:  
- `Type or value exists (20)` → already a member ✅  
- `No such attribute: member` → group is `posixGroup`. Use instead:  

```ldif
dn: cn=jenkins-admins,ou=groups,dc=nakodtech,dc=com
changetype: modify
add: memberUid
memberUid: adminuser1
```  

👉 Set Jenkins group filter → `memberUid={1}`  

---

## 4️⃣ Verify Setup  
✔️ Confirm DN:  
```bash
ldapsearch -x -LLL -b "dc=nakodtech,dc=com" "(uid=adminuser1)" dn
```  

✔️ Test login with `adminpass1`:  
```bash
ldapwhoami -x -D "uid=adminuser1,ou=users,dc=nakodtech,dc=com" -W
```  

✔️ Check group:  
```bash
ldapsearch -x -LLL -b "cn=jenkins-admins,ou=groups,dc=nakodtech,dc=com" member
```  

---

## 5️⃣ Jenkins Refresh  
⚙️ In **Matrix Security** → grant **Administer** to `jenkins-admins`.  

Login:  
- 👤 Username → `adminuser1`  
- 🔑 Password → `adminpass1`  

If perms missing →  
1. 🔄 Save **Global Security** again.  
2. 🗑️ Delete cached user under **People** → re-login.  

✨ Done → `adminuser1` is now an **Admin** 🎉  
