# 🧭 Jenkins — Pipeline from SCM (Git) Notes

## 🚀 Goal

Run a **Pipeline job** whose Jenkinsfile lives in your Git repo.

---

## ⚙️ Quick Setup (Pipeline → *Pipeline script from SCM*)

1. **SCM:** `Git`
2. **Repository URL:** HTTPS or SSH URL of the repo
3. **Credentials:**

   * Public repo → `- none -`
   * Private repo → add credentials:

     * *HTTPS:* **Username with password/token** (GitHub PAT)
     * *SSH:* **SSH Username with private key** (e.g., `git` + key)
4. **Branches to build:** `main` (or `*/main`, or `*/feature/*`)
5. **Script Path:** `Jenkinsfile` (or `ci/Jenkinsfile` if stored elsewhere)
6. *(Optional)* **Lightweight checkout:** ✅ fetch only the Jenkinsfile via API (faster). Uncheck if you need `.git` metadata during Pipeline.
7. **Save** → Build.

---

## 🧩 Field-by-field

* **Repository URL**

  * *HTTPS:* `https://github.com/<user>/<repo>.git`
  * *SSH:* `git@github.com:<user>/<repo>.git`
* **Credentials**

  * Add via *Manage Jenkins → Credentials → (Global)* → **Add Credentials**.
  * Scope: Global; ID: short, memorable (e.g., `gh-pat` / `gh-ssh`).
* **Branches to build**

  * Single branch: `main`
  * Any branch: `**`
  * Patterns: `origin/release/*` or `*/PR-*` (with proper refspec).
* **Script Path**

  * Default `Jenkinsfile`. Use `subdir/Jenkinsfile` if stored in a folder.
* **Repository browser**: optional; helps linkify changes in the UI.

---

## 🧰 Advanced (Git → *Additional Behaviours*)

* **Shallow clone:** set depth (e.g., 10) to speed up.
* **Sparse checkout:** only pull certain paths.
* **Clean before/after checkout:** ensures reproducible workspaces.
* **Custom refspec:** needed for building PR refs (e.g., `+refs/pull/*:refs/remotes/origin/pr/*`).
* **Checkout to a sub-directory:** keep workspace tidy.

---

## 🔔 Triggers

* **GitHub hook trigger for GITScm polling** (preferred) + set a webhook in GitHub → `<JENKINS_URL>/github-webhook/`.
* Or **Poll SCM** schedule: `H/2 * * * *` (every \~2 min).

---

## 🧪 Pipeline snippets

**Minimal Declarative Jenkinsfile in repo:**

```groovy
pipeline {
  agent { label 'agent-1' }
  tools { maven 'Maven-3.9.11'; jdk 'jdk17' }
  stages {
    stage('Checkout') { steps { checkout scm }}
    stage('Build')    { steps { sh 'mvn -B -U clean package' } }
  }
}
```

**Explicit Git checkout (useful when not using `checkout scm`):**

```groovy
stage('Checkout') {
  steps {
    git branch: 'main', url: 'https://github.com/OWNER/REPO.git', credentialsId: 'gh-pat'
  }
}
```

---

## 🔒 Security & Access

* Use **PAT with least privileges** (repo read) or **deploy keys** for SSH.
* Avoid storing secrets in Jenkinsfile; use **Credentials Binding**.
* If using SSH, set **Host Key Verification Strategy** to known hosts (not “Non-verifying”).

---

## 🧠 Multibranch vs Pipeline-from-SCM

* **Pipeline-from-SCM:** single job that always builds a chosen branch.
* **Multibranch Pipeline:** auto-discovers branches/PRs with their Jenkinsfiles; recommended for active repos.

---

## 🩺 Troubleshooting

* **`Jenkinsfile not found`** → wrong *Script Path* or branch.
* **`Host key verification failed`** → fix SSH known hosts / set verification strategy.
* **`Authentication failed`** → wrong credentials or missing PAT scopes (`repo` for GitHub).
* **`mvn: not found`** → add `tools { maven 'Maven-3.9.11'; jdk 'jdk17' }` or use `./mvnw`.
* **Java mismatch (`JCTree qualid`)** → project requires **JDK 11/17**; select matching JDK in job tools or update POM/plugins.

---

## ✅ Best Practices

* Keep Jenkinsfile at repo root (`/Jenkinsfile`).
* Enable **Lightweight checkout** unless you need `.git` data.
* Use **shallow clone** for speed; clean workspace for reproducibility.
* Prefer webhooks over polling.
* For complex checkout logic, consider a **Multibranch Pipeline**.
