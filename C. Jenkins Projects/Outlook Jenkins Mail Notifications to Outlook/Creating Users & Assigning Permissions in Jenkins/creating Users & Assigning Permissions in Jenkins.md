# 👤 Creating Users & Assigning Permissions in Jenkins  

---

## 📝 Step 1: Create a User  
1. 🔑 **Log in as Admin**  
   (you need Admin rights).  
2. ⚙️ Go to → **Manage Jenkins → Security → Manage Users**  
3. ➕ Click **Create User** → fill in:  
   - 👤 Username  
   - 🔒 Password  
   - 🏷️ Full Name  
   - 📧 Email  
4. 💾 Save → ✅ user created  

⚠️ By default, without Authorization strategy → **every user = Admin**  

---

## 🔑 Step 2: Configure Permissions  

### 🟦 Option A — Matrix-based Security (Global)  
- ⚙️ **Manage Jenkins → Configure Global Security**  
- Select **Matrix-based security**  
- ➕ Add user IDs (created in Jenkins or LDAP)  
- ☑️ Assign permissions with checkboxes (Read, Build, Configure, Admin)  

---

### 🟩 Option B — Project-based Matrix Security  
- 📂 Go to **Job → Configure → Enable project-based security**  
- 🎯 Assign permissions **only for that job/project**  
- Example: 👨‍💻 `bob` can build **Project A** but not access **Project B**  

---

### 🟨 Option C — Role-based Strategy  
- 📦 Install **Role Strategy Plugin**  
- 🛠️ Define roles → (Admin / Developer / Viewer)  
- 👥 Map users or groups to roles  

---

### 🟥 Option D — LDAP / SSO  
- 🌐 If integrated → users come from LDAP/AD/GitHub  
- 🔗 Jenkins only maps **roles/permissions** to external accounts  

---

## 🆔 Which User ID to Use?  
- 🔹 **Jenkins DB** → use the **username** you created in **Manage Users**  
- 🔹 **LDAP/SSO** → use the **external username** (e.g., LDAP UID)  

---

## ⚡ Key Points  
✅ You can add users directly in Jenkins UI  
🚫 Default = **Admin access** (not safe)  
🛠 Use **Matrix** or **Role Strategy** for fine-grained control  
🔐 LDAP/SSO is optional (for external identity management)  
