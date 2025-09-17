# 👨‍💻 Create `devuser1` & Add to `jenkins-devs` (LDAP + Jenkins)

---

## 1️⃣ Generate Password Hash  
🔑 Hash password `devpass1`  

```bash
slappasswd -h {SSHA} -s devpass1
```  

👉 Copy the full output (e.g., `{SSHA}xyz...=`)  

---

## 2️⃣ Create the User  
📂 **devuser1.ldif**  

```ldif
dn: uid=devuser1,ou=users,dc=nakodtech,dc=com
objectClass: inetOrgPerson
uid: devuser1
sn: Developer1
cn: Dev User1
userPassword: {SSHA}PASTE_HASH_FROM_SLAPPASSWD_HERE
```  

🚀 Add user:  
```bash
ldapadd -x -D "cn=admin,dc=nakodtech,dc=com" -W -f devuser1.ldif
```  

⚡ If user exists → reset password:  

📂 **reset-devuser1-pass.ldif**  
```ldif
dn: uid=devuser1,ou=users,dc=nakodtech,dc=com
changetype: modify
replace: userPassword
userPassword: {SSHA}PASTE_HASH_FROM_SLAPPASSWD_HERE
```  

```bash
ldapmodify -x -D "cn=admin,dc=nakodtech,dc=com" -W -f reset-devuser1-pass.ldif
```  

---

## 3️⃣ Add User to Group  
📂 **add-devuser1-to-devs.ldif**  

```ldif
dn: cn=jenkins-devs,ou=groups,dc=nakodtech,dc=com
changetype: modify
add: member
member: uid=devuser1,ou=users,dc=nakodtech,dc=com
```  

🚀 Apply:  
```bash
ldapmodify -x -D "cn=admin,dc=nakodtech,dc=com" -W -f add-devuser1-to-devs.ldif
```  

⚠️ Errors:  
- `Type or value exists (20)` → already a member ✅  
- `No such attribute: member` → group is `posixGroup`. Use instead:  

```ldif
dn: cn=jenkins-devs,ou=groups,dc=nakodtech,dc=com
changetype: modify
add: memberUid
memberUid: devuser1
```  

👉 Set Jenkins group filter → `memberUid={1}`  

---

## 4️⃣ Verify Setup  
✔️ Confirm DN:  
```bash
ldapsearch -x -LLL -b "dc=nakodtech,dc=com" "(uid=devuser1)" dn
```  

✔️ Test login with `devpass1`:  
```bash
ldapwhoami -x -D "uid=devuser1,ou=users,dc=nakodtech,dc=com" -W
```  

✔️ Check group:  
```bash
ldapsearch -x -LLL -b "cn=jenkins-devs,ou=groups,dc=nakodtech,dc=com" member
```  

---

## 5️⃣ Jenkins Refresh  
⚙️ In **Matrix Security** → grant **Build/Read** to `jenkins-devs`.  

Login:  
- 👤 Username → `devuser1`  
- 🔑 Password → `devpass1`  

If perms missing →  
1. 🔄 Save **Global Security** again.  
2. 🗑️ Delete cached user under **People** → re-login.  

✨ Done → `devuser1` is now part of **jenkins-devs** 🎉  
