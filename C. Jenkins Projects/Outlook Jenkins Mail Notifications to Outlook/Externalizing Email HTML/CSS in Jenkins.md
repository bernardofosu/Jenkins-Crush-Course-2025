# ✉️ Externalizing Email HTML/CSS in Jenkins (with Emojis)

You can (and usually should) move the **HTML/CSS** out of the **Jenkinsfile** and load it at runtime. Here are **three clean ways**, plus working snippets.

---

## 1) 📁 Read files from the repo (simplest)

Put your templates in the repo, e.g.:

```
repo/
└─ email/
   ├─ body.html    # uses template placeholders
   └─ styles.css
```

### `body.html` (template)

```html
<html>
  <head>
    <style>
      ${css}  <!-- we’ll inject styles.css into this variable -->
    </style>
  </head>
  <body>
    <div style="border: 4px solid ${bannerColor}; padding: 10px;">
      <h2>${jobName} - Build ${buildNumber}</h2>
      <div style="background-color: ${bannerColor}; padding: 10px;">
        <h3 style="color: white;">Pipeline Status: ${pipelineStatus}</h3>
      </div>
      <p>Check the <a href="${buildUrl}">console output</a>.</p>
    </div>
  </body>
</html>
```

### `Jenkinsfile` (Declarative)

```groovy
pipeline {
  agent any

  stages {
    stage('Build') { steps { echo 'Build...' } }
  }

  post {
    always {
      script {
        // 1) Compute values
        def jobName        = env.JOB_NAME
        def buildNumber    = env.BUILD_NUMBER
        def pipelineStatus = (currentBuild.result ?: 'UNKNOWN').toUpperCase()
        def bannerColor    = (pipelineStatus == 'SUCCESS') ? 'green' : 'red'
        def buildUrl       = env.BUILD_URL

        // 2) Load files from workspace
        def rawHtml = readFile('email/body.html')
        def css     = readFile('email/styles.css')

        // 3) Fill template using a Groovy template engine
        def binding = [
          jobName: jobName,
          buildNumber: buildNumber,
          pipelineStatus: pipelineStatus,
          bannerColor: bannerColor,
          buildUrl: buildUrl,
          css: css
        ]
        def engine = new groovy.text.StreamingTemplateEngine()
        def body   = engine.createTemplate(rawHtml).make(binding).toString()

        // 4) Send email
        emailext(
          subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
          body: body,
          mimeType: 'text/html',
          to: 'ofosubernard848@outlook.com',
          from: 'ofosubernard848@outlook.com',
          replyTo: 'ofosubernard848@outlook.com'
        )
      }
    }
  }
}
```

---

## 2) 🧩 Jenkins **Shared Library** resources (reusable across many jobs)

* Put `vars/` and `resources/` in a **shared library** repo.
* Store `resources/email/body.html` and `resources/email/styles.css`.
* Load with `libraryResource`.

```groovy
@Library('your-shared-lib') _
pipeline {
  agent any
  post {
    always {
      script {
        def rawHtml = libraryResource('email/body.html')
        def css     = libraryResource('email/styles.css')
        def engine  = new groovy.text.StreamingTemplateEngine()
        def body    = engine.createTemplate(rawHtml).make([
          jobName: env.JOB_NAME,
          buildNumber: env.BUILD_NUMBER,
          pipelineStatus: (currentBuild.result ?: 'UNKNOWN').toUpperCase(),
          bannerColor: ((currentBuild.result ?: '') == 'SUCCESS') ? 'green' : 'red',
          buildUrl: env.BUILD_URL,
          css: css
        ]).toString()

        emailext(subject: "Mail", body: body, mimeType: 'text/html', to: 'ofosubernard848@outlook.com')
      }
    }
  }
}
```

---

## 3) 🧰 **Config File Provider** plugin (managed templates in Jenkins UI)

* Upload your HTML/CSS as managed files in **Manage Jenkins → Managed files**.
* Use `configFileProvider` to access them by `fileId`.

```groovy
pipeline {
  agent any
  post {
    always {
      script {
        configFileProvider([
          configFile(fileId: 'html-template-id', variable: 'HTML_PATH'),
          configFile(fileId: 'css-template-id',  variable: 'CSS_PATH')
        ]) {
          def rawHtml = readFile(HTML_PATH)
          def css     = readFile(CSS_PATH)
          def engine  = new groovy.text.StreamingTemplateEngine()
          def body    = engine.createTemplate(rawHtml).make([
            jobName: env.JOB_NAME,
            buildNumber: env.BUILD_NUMBER,
            pipelineStatus: (currentBuild.result ?: 'UNKNOWN').toUpperCase(),
            bannerColor: ((currentBuild.result ?: '') == 'SUCCESS') ? 'green' : 'red',
            buildUrl: env.BUILD_URL,
            css: css
          ]).toString()

          emailext(subject: "Mail", body: body, mimeType: 'text/html', to: 'ofosubernard848@outlook.com')
        }
      }
    }
  }
}
```

---

## 💡 Important tips for email HTML/CSS

* ✅ **Inline styles** are the most reliable across email clients. External `<link rel="stylesheet">` is often ignored/blocked. If you keep a separate `styles.css`, inject it into a `<style>${css}</style>` tag (as shown).
* ✅ Use a **template engine** to replace placeholders. Simply calling `readFile('body.html')` won’t interpolate `${vars}`; you need `StreamingTemplateEngine` (or `SimpleTemplateEngine`).
* ✅ Keep images as **absolute URLs** or embed as **Base64** if your mail client supports it.

✨ This approach keeps your Jenkinsfile **clean**, makes templates **reusable**, and emails **consistent** across jobs.
