# 📧 Jenkins Inbuilt Email Notification with Gmail

---

## 🖥️ Step 1: Enable Gmail App Password

1. Go to 👉 [Google Account Security](https://myaccount.google.com/security).
2. Make sure **2-Step Verification** is ON ✅.
3. Generate an **App Password** 🔑:

   * Go to **App passwords**.
   * Select: Mail 📩 → Other (custom name e.g., Jenkins).
   * Copy the **16-character app password** (e.g., `abcd efgh ijkl mnop`).

---

## ⚙️ Step 2: Jenkins SMTP Configuration

1. In Jenkins → **Manage Jenkins → Configure System** ⚙️.

2. Scroll to **E-mail Notification** 📬.

3. Fill in:

   * **SMTP Server:** `smtp.gmail.com`
   * **Port:** `587`
   * ✅ Use TLS
   * ✅ Use SMTP Authentication
   * **User Name:** `ofosubernard848@gmail.com`
   * **Password:** *App Password from Step 1* 🔑
   * **Default user e-mail suffix:** `@gmail.com`

4. Save ✅.

---

## 🧪 Step 3: Test Email in Jenkins

1. Scroll down → **Test configuration by sending test e-mail**.
2. Enter: `ofosubernard848@gmail.com`.
3. Click **Test** 🚀.
4. If successful → you’ll see a test email in your inbox 🎉.

---

## 📝 Step 4: Basic Pipeline Example

Use the inbuilt `mail` step:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "Building..."
            }
        }
    }

    post {
        always {
            mail to: 'ofosubernard848@gmail.com',
                 subject: "Build #${env.BUILD_NUMBER} - ${currentBuild.result}",
                 body: "Hello Bernard 👋, check Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
```

📩 **Result:**
You’ll get a simple plain-text email:

* **Subject:** Build #12 - SUCCESS
* **Body:** Hello Bernard 👋, check Jenkins for details: [http://jenkins/job/my-pipeline/12/](http://jenkins/job/my-pipeline/12/)

---

✨ That’s it! Now Jenkins can send Gmail alerts using the built-in plugin without installing anything extra.


## 🛠️ Fix Jenkins "address not configured yet" Issue

When Jenkins emails show **“address not configured yet”**, it means the **System Admin E-mail Address** is not set in Jenkins.

---

### ✅ Steps to Fix

1️⃣ **Go to Manage Jenkins** ⚙️
2️⃣ **Click Configure System** 🖥️
3️⃣ Scroll to **Jenkins Location** 🌍
4️⃣ Fill in:

* **Jenkins URL** → `http://34.229.81.126:8080/` 🌐
* **System Admin e-mail address** → `ofosubernard848@gmail.com` 📧
  5️⃣ **Save** ✅

---

### 📧 Before Fix

```
From: address not configured yet <nobody@nowhere>
```

### 📧 After Fix

```
From: Bernard Ofosu <ofosubernard848@gmail.com>
```

---

✨ Now Gmail will correctly display your name and email instead of the pl
