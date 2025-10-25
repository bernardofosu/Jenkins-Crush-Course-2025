# 🔐 Jenkins Role Strategy + LDAP — Battle‑Tested Checklist

Make permissions clean, scalable, and safe using **Role‑Based Strategy** with **LDAP groups**.

---

## 1) 📦 Install & Enable

* **Manage Jenkins → Manage Plugins → Available** → install **Role‑based Authorization Strategy** (aka *Role Strategy Plugin*).
* **Manage Jenkins → Security (Configure Global Security)** → **Authorization:** select **Role‑Based Strategy** → **Save**.

> 🔗 Direct pages (handy):
>
> * **Manage Roles:** `/role-strategy/manage-roles`
> * **Assign Roles:** `/role-strategy/assign-roles`

---

## 2) 🧩 Define Global Roles  *(Manage → Manage and Assign Roles → Manage Roles)*

Create these starter roles (tweak as needed):

### 👑 `admin`

* **Overall → Administer** *(implies Read)*
* (Optional) **Credentials → ManageDomains / View**

### 🧑‍💻 `dev`

* **Overall → Read**
* **Job → Read, Build**
* (Optional) **Credentials → View**, **View → Read**

### 👀 `viewer`

* **Overall → Read**
* **Job → Read**
* **View → Read**

> 🗂️ **Project roles (optional):** Use regex patterns to scope by folder/job names.
>
> * Example: **`teamA-dev`** with pattern `^teamA-.*` → same perms as `dev` but only for Team A jobs.

---

## 3) 👥 Assign Roles to LDAP Groups  *(Manage → Manage and Assign Roles → Assign Roles)*

Under **Global roles**:

* **Add Group** `jenkins-admins` → ✅ **admin**
* **Add Group** `jenkins-devs` → ✅ **dev**
* **Add Group** `jenkins-viewers` → ✅ **viewer**
* **Add User** `adminuser` (break‑glass) → ✅ **admin**

**Notes:**

* Type the **exact LDAP CN** for groups (no `@`, no `GROUP:`). Jenkins resolves them via LDAP.
* Keep one explicit **break‑glass user** in `admin` so you’re never locked out.

---

## 4) 🔎 Verify Group Mapping Works

1. Log out/in as: `adminuser1`, `devuser1`, `viewer1`.
2. Open **`/whoAmI`** → confirm **Authorities** include your LDAP group(s).
3. If user authenticates but has no perms, recheck LDAP:

   * **Group membership strategy:** *Search for LDAP groups containing user*
   * **Filter (groupOfNames + DN membership):**

     ```
     (member={0})
     ```

     *(or explicit)* `(&(objectClass=groupOfNames)(member={0}))`
   * **posixGroup + memberUid:** use `**(memberUid={1})**` instead.
   * **User base:** `ou=users` · **Group base:** `ou=groups` · **Manager DN:** `cn=admin,dc=nakodtech,dc=com`
4. **Save**, then re‑login (or delete cached user under **People**) and retest.

---

## 5) 🛡️ Hardening Tips

* 🚫 Leave **Anonymous** with **no** permissions.
* 🧯 Keep **one explicit admin user** assigned to **admin** (break‑glass).
* 💾 Back up `/var/lib/jenkins/config.xml` after a good setup.
* 🧱 Prefer **Project roles + regex** for team scoping (e.g., `^mobile-.*`, `^data/.*`).

---

## 🧭 Quick “Starter” Layout

**Manage Roles → Global roles**

* `admin` → **Overall/Administer**
* `dev` → **Overall/Read**, **Job/Read**, **Job/Build** *(+ Credentials/View optional)*
* `viewer` → **Overall/Read**, **Job/Read**

**Assign Roles → Global roles**

* `jenkins-admins` → **admin**
* `jenkins-devs` → **dev**
* `jenkins-viewers` → **viewer**
* `adminuser` (user) → **admin** *(break‑glass)*

---

## ❓ FAQ

* **Where is “Manage and Assign Roles”?** Appears only **after** you select **Role‑Based Strategy**. Use the direct URLs above if you don’t see the menu yet.
* **Does this replace LDAP?** No. Both Matrix and Role‑Based use your **LDAP Security Realm**; Role‑Based just organizes permissions via roles.
* **Why don’t groups auto‑appear?** You must **Add Group** by name; Jenkins resolves membership at login.

---

### ✅ Result

Clean, scalable permissions: add people to LDAP groups → they inherit the right Jenkins access automatically. 🎯



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
````
[LDAP Testing](ldap_testing_cheatsheet.md)

## ❓ FAQ

* **Where is “Manage and Assign Roles”?** Appears only **after** you select **Role‑Based Strategy**. Use the direct URLs above if you don’t see the menu yet.
* **Does this replace LDAP?** No. Both Matrix and Role‑Based use your **LDAP Security Realm**; Role‑Based just organizes permissions via roles.
* **Why don’t groups auto‑appear?** You must **Add Group** by name; Jenkins resolves membership at login.

---

### ✅ Result

Clean, scalable permissions: add people to LDAP groups → they inherit the right Jenkins access automatically. 🎯

---

# 🆚 Role‑Based Strategy vs Matrix (with LDAP)

Both use your **LDAP Security Realm** for identities (users & groups). They differ in **how permissions are assigned**.

| Aspect                  | 🧮 Matrix‑based (built‑in)                                | 🧩 Role‑Based Strategy (plugin)                                  |
| ----------------------- | --------------------------------------------------------- | ---------------------------------------------------------------- |
| **How it works**        | Grant permissions directly to users/groups in a big grid. | Define **roles** (bundles), assign them to users/groups.         |
| **Scale**               | OK small; becomes click‑heavy/messy as instance grows.    | **Scales well**; change role once → applies everywhere.          |
| **Project scoping**     | Limited; per‑item entries get painful.                    | **Project roles + regex** (e.g., `^teamA-.*`) for clean scoping. |
| **Maintainability**     | Many checkboxes; easy to mis‑configure.                   | Centralized roles = fewer mistakes, easier reviews.              |
| **Least privilege**     | Possible but error‑prone at scale.                        | Easier to enforce (admin/dev/viewer baselines).                  |
| **Lockout risk**        | Higher (one wrong grid change).                           | Lower (keep a break‑glass admin role).                           |
| **Built‑in vs plugin**  | Built‑in.                                                 | Requires **Role Strategy** plugin (keep updated).                |
| **Auditing/compliance** | Hard to reason about large grids.                         | Roles map to policy; simpler audits.                             |
| **Rename churn**        | Painful; lots of manual edits.                            | Roles & regex patterns survive renames better.                   |

## ✅ Recommendation (production)

* Use **Role‑Based Strategy + LDAP groups** for anything beyond a tiny instance.
* Global roles: **admin**, **dev**, **viewer**.  Optional project roles with patterns.
* Map groups → roles: `jenkins-admins → admin`, `jenkins-devs → dev`, `jenkins-viewers → viewer`.
* Keep one **break‑glass LDAP user** in **admin**.

## 🔒 Important reminders

* **Authorization (Matrix/Role)** ≠ **Authentication (LDAP/Jenkins DB)**. Switching strategies does **not** change login method. The one‑time `initialAdminPassword` never “comes back”.
* LDAP group filter must be valid: `**(member={0})**` (for `groupOfNames` + `member: <user DN>`). If using `posixGroup` + `memberUid`, use `**(memberUid={1})**`.
* After changes: **Save**, log out/in, check **`/whoAmI`** to confirm authorities. Avoid giving **Anonymous** permissions in prod.
