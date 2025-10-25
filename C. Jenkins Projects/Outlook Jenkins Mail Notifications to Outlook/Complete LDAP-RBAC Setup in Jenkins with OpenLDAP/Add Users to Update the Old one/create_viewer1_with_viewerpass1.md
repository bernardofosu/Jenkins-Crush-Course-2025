# 👀 Creating viewer1 in OpenLDAP and Adding to jenkins-viewers Group

---

## 1️⃣ Generate Password Hash

Create a secure hash for the password **viewerpass1**:

```bash
slappasswd -h {SSHA} -s viewerpass1
# copy the full output, e.g. {SSHA}xyz...=
```

---

## 2️⃣ Create the User Entry

📂 **viewer1.ldif**

```ldif
dn: uid=viewer1,ou=users,dc=nakodtech,dc=com
objectClass: inetOrgPerson
uid: viewer1
sn: Viewer1
cn: Viewer User1
userPassword: {SSHA}PASTE_HASH_FROM_SLAPPASSWD_HERE
```

👉 Load into LDAP:

```bash
ldapadd -x -D "cn=admin,dc=nakodtech,dc=com" -W -f viewer1.ldif
```

---

## 3️⃣ Add viewer1 to jenkins-viewers Group

📂 **add-viewer1-to-viewers.ldif**

```ldif
dn: cn=jenkins-viewers,ou=groups,dc=nakodtech,dc=com
changetype: modify
add: member
member: uid=viewer1,ou=users,dc=nakodtech,dc=com
```

👉 Apply it:

```bash
ldapmodify -x -D "cn=admin,dc=nakodtech,dc=com" -W -f add-viewer1-to-viewers.ldif
```

⚠️ If you get `Type or value exists (20)` → already a member (OK).  
⚠️ If you get `No such attribute: member` → group is `posixGroup`. Use `memberUid` instead:

```ldif
dn: cn=jenkins-viewers,ou=groups,dc=nakodtech,dc=com
changetype: modify
add: memberUid
memberUid: viewer1
```

And set Jenkins to use `memberUid={1}`.

---

## 4️⃣ Verify

✅ Confirm DN exists:

```bash
ldapsearch -x -LLL -b "dc=nakodtech,dc=com" "(uid=viewer1)" dn
```

✅ Test bind with cleartext password:

```bash
ldapwhoami -x -D "uid=viewer1,ou=users,dc=nakodtech,dc=com" -W
```

✅ Confirm group membership:

```bash
ldapsearch -x -LLL -b "cn=jenkins-viewers,ou=groups,dc=nakodtech,dc=com" member
```

---

## 5️⃣ Jenkins Configuration

* Ensure **Matrix-based Security** grants **Read-only** to group `jenkins-viewers`.
* Log in with:

```
Username: viewer1
Password: viewerpass1
```

---

# ✨ Summary

👤 **viewer1** = read‑only Jenkins user  
👥 Group = `jenkins-viewers`  
🔑 Password = `viewerpass1`  
📌 Role = View jobs, no build/admin rights  
