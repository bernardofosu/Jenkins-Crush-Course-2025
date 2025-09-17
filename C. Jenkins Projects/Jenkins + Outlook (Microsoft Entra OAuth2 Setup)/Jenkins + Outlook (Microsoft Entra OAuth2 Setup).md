# 📧 Jenkins + Outlook (Microsoft Entra OAuth2 Setup)

---

## 🖥️ Step 1: Log into Microsoft Entra

🔗 Go to [https://entra.microsoft.com](https://entra.microsoft.com)
➡️ Sign in with your Outlook account (`ofosubernard848@outlook.com`).

---

## 🏗️ Step 2: Register App in Entra ID

1. Navigate → **Entra ID → App registrations → + New registration**
2. Fill in:

   * **Name** → `Jenkins-Mail-Notifier` 📨
   * **Supported accounts** → Accounts in any directory + personal accounts
   * **Redirect URI** → `https://localhost` 🌐 (temporary)
3. ✅ Grant admin consent to **openid** & **offline\_access**
4. Click **Register**

---

## 🔑 Step 3: Collect App IDs

From **Overview**:

* **Client ID** 🆔 → ``
* **Tenant ID** 🏢 → ``

👉 Save these values!

---

## 🔐 Step 4: Create a Client Secret

1. Go to → **Certificates & secrets → + New client secret**
2. Add description → `JenkinsSecret`
3. Expiry → 6 or 12 months
4. ⚠️ Copy the **Value** immediately (you won’t see it again)

---

## 📜 Step 5: Add API Permissions

1. Go to **API permissions → + Add permission**
2. Choose **Microsoft Graph → Delegated permissions**
3. Tick ✅:

   * `SMTP.Send` (send via SMTP AUTH)
   * `offline_access` (refresh tokens)
   * `openid` (basic identity)
4. Click **Add permissions**
5. ✅ Click **Grant admin consent** for your tenant

---

## 🔁 Step 6: Get OAuth2 Tokens

### (A) Request Device Code

```bash
curl -sS -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=<client-id>" \
  -d "scope=offline_access openid https://graph.microsoft.com/SMTP.Send" \
  "https://login.microsoftonline.com/634cfbbb-ded3-4772-b444-5d0017142b51/oauth2/v2.0/devicecode"
```

## 👉 Copy `verification_uri` + `user_code` → Open in browser → Sign in.

### 🖥️ How to sign in with the device code

1️⃣ Copy the verification URI from the output:
```sh
👉 https://microsoft.com/devicelogin
```

2️⃣ Open it in your browser 🌐 (any browser is fine).

3️⃣ Paste the user_code shown in your terminal:
```s
👉 DX
```

4️⃣ It will prompt you to sign in. Use your Microsoft account:
```sh
📧 ofosubernard848@outlook.com
```

5️⃣ After you sign in successfully, the browser will confirm ✅.

### (B) Exchange Code for Tokens

```bash
curl -sS -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=urn:ietf:params:oauth:grant-type:device_code" \
  -d "client_id=<client-id>" \
  -d "device_code=<YOUR_DEVICE_CODE>" \
  "https://login.microsoftonline.com/634cfbbb-ded3-4772-b444-5d0017142b51/oauth2/v2.0/token"
```

👉 Response includes:

* `access_token` (short-lived)
* `refresh_token` 🔑 (long-lived → save this!)



## ⚙️ Step 7: Configure Jenkins

Go to → **Manage Jenkins → Configure System → Extended E-mail Notification**

Fill in:

* **SMTP Server** → `smtp.office365.com`
* **Port** → `587`
* ✅ Use TLS

**Authentication → OAuth2.0**:

* Client ID → `client-id`
* Client Secret → `<YOUR_CLIENT_SECRET_VALUE>`
* Token URL →
  `https://login.microsoftonline.com/634cfbbb-ded3-4772-b444-5d0017142b51/oauth2/v2.0/token`
* Refresh Token → `<YOUR_REFRESH_TOKEN>`
* Scope → `https://graph.microsoft.com/SMTP.Send offline_access openid`

---

## 🚀 Step 8: Test It

1. Use **Send Test E-mail** from Jenkins config
2. Or run your pipeline → Jenkins will send via Outlook 📧

✅ Now Jenkins is using Microsoft Entra OAuth2 (modern auth) with Outlook.
