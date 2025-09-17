# 📧 Jenkins Email Plugins

---

## 1️⃣ E-mail Notification (basic, built-in)

🏗️ Comes bundled with Jenkins.

📩 Sends **plain, simple emails** via SMTP (no fancy formatting).

⚙️ Config: **Manage Jenkins → Configure System → E-mail Notification**.

🔑 Uses **fixed recipients only** (e.g., `someone@example.com`).

📜 Emails are **plain text only** (❌ no HTML, no attachments).

⚠️ Very limited:

* Can’t control triggers per stage.
* Can’t format email body.
* Can’t send build logs or custom reports.

👉 Best for **just basic alerts** like “Build failed!”.

### Example

```groovy
mail to: 'devs@example.com',
     subject: "Build #${env.BUILD_NUMBER} - ${currentBuild.result}",
     body: "Check Jenkins for details: ${env.BUILD_URL}"
```

📩 Output (plain text):

```
Subject: Build #15 - SUCCESS
Body:
Check Jenkins for details: http://jenkins.example.com/job/my-pipeline/15/
```

---

## 2️⃣ Extended E-mail Notification (Email Extension Plugin, aka emailext)

📦 Comes from the **Email Extension Plugin** (requires installation).

🎨 Allows **custom subject, body, recipients, HTML, and templates**.

⚙️ Config: **Manage Jenkins → Configure System → Extended E-mail Notification**.

🖋️ Use in pipelines with the **`emailext {}` step**.

🔧 Can use **Groovy / Jelly templates** → dynamic emails (job name, build URL, stage results).

📜 Supports **HTML emails, attachments, reports, logs**.

🎯 Flexible triggers:

* Only on failure ❌
* On success ✅
* On unstable ⚠️
* Always 🔁

📊 Can create **status dashboards** inside the email.

👉 Best for **production**: rich emails, CI/CD summaries, multiple recipients.

### Example

```groovy
emailext(
  subject: "📢 ${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${currentBuild.result}",
  body: """
    <h2>Build Summary</h2>
    <p>Status: <b>${currentBuild.result}</b></p>
    <p>Console: <a href="${env.BUILD_URL}">Logs</a></p>
  """,
  to: 'devs@example.com',
  mimeType: 'text/html'
)
```

📧 Output (HTML email):

* ✅ Green/Red status banner
* 📝 Build number, job name
* 🔗 Link to console logs

---

# 🔑 In Short

* **E-mail Notification** = 🪶 lightweight, plain, old-school.
* **Extended E-mail Notification** = 💪 powerful, customizable, HTML-rich (used in prod).

👉 Since modern providers (Outlook/Gmail) require OAuth2 and rich formatting → use **`emailext`** for real-world pipelines.
