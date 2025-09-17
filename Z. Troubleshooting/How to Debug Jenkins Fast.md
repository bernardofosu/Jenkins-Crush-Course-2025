# 🧭 How to Debug Jenkins Fast

**Golden rule:** Open the build ➜ **Console Output** and read **bottom → up** until you hit the *first* clear error line. Copy that line (or screenshot) if you need help. 🧠

---

## 💡 Quick Triage Flow

1. 🔎 **Find failing stage/step** in the stage view.
2. 📜 Open **Console Output**, scroll to the **first** error.
3. 🏷️ Note the **exit code** (e.g., 127, 137, 403/500 HTTP).
4. 🧪 Re-run with more logs if useful (e.g., `mvn -X`, `npm --verbose`).
5. 🖥️ Check **agent**: online? labels match? tools available? PATH?
6. 🧹 If messy: `cleanWs()` then rebuild.

---

## 🔐 Git/Checkout Problems

* **128** ➜ generic Git failure (bad URL/branch, DNS, network).
* **403 Unauthorized** ➜ wrong/expired **credentials** or token scopes.
* **Repo/branch not found** ➜ URL/branch typo; confirm branch exists.
* **Fixes**:

  * Use **credentialsId** for private repos.
  * For Multibranch jobs: `checkout scm` (don’t hardcode branch).
  * Behind proxy? Configure **Manage Jenkins → Plugins → Advanced** proxy.
  * Verify Git on agent: `git --version`.

**Clone to subfolder** (avoid double checkouts):

```groovy
options { skipDefaultCheckout(true) }
stage('Checkout') {
  steps {
    checkout([$class: 'GitSCM',
      branches: [[name: '*/main']],
      userRemoteConfigs: [[url: 'https://…', credentialsId: 'git-cred']],
      extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'repo-name']]
    ])
  }
}
```

---

## 🧱 Common Exit Codes & Meanings

* **127** ➜ `command not found` (e.g., `mvn` not installed / not on PATH).
* **137** ➜ Out Of Memory / killed by OS (`SIGKILL`). Increase RAM / heap / container limits.
* **1** ➜ Generic script failure; read the immediate error line(s).
* **143** ➜ Terminated (`SIGTERM`)—timeout or manual abort.

---

## 🛠️ Tools Not Found

* `mvn not found`, `java: command not found` ➜ Install tools or declare in pipeline:

```groovy
pipeline {
  agent any
  tools { maven 'maven3'; jdk 'jdk17' }
  stages { stage('Verify') { steps { sh 'java -version; mvn -v' } } }
}
```

* **Node/Python**: use tool installers or ensure PATH on agent.

---

## 🗂️ Workspace Issues

* Wrong folder ➜ remember checkout goes to **\$WORKSPACE** by default. If you cloned to a subfolder, run build **inside** it:

```groovy
dir('repo-name/dir') { sh 'mvn -B -ntp clean package' }
# or: sh 'mvn -B -ntp -f repo-name/dir/pom.xml clean package'
```

* Clean between builds: `post { always { cleanWs() } }`.

---

## 🧩 `changeset` Conditions (stage gating)

* `when { changeset }` ➜ run stage only if **any** files changed.
* `when { changeset pattern: 'src/**', comparator: 'GLOB' }` ➜ run if **matching** files changed.
* ❗ Needs checkout **before** the stage; empty on the first build.

---

## 📡 Network/Proxy/SSL

* `Could not resolve host` ➜ DNS/proxy. Try `curl -I https://github.com` on agent.
* Corporate proxy? Set proxy in Jenkins global config and environment.

---

## 🔍 SonarQube, Nexus & Friends

* **Sonar analysis** fails: server URL/token wrong, server down, or plugin missing.

  * Check **Sonar server logs**, project key, and token scopes.
  * Jenkins: install **SonarQube Scanner** plugin and set **Server** + **Token**.
* **HTTP 500** (Nexus/Sonar) ➜ check service logs; restart service/pod; check disk/DB.

---

## 🧪 Handy Debug Snippets

```groovy
sh 'whoami; id -a'
sh 'echo WORKSPACE=$WORKSPACE; pwd; ls -la'
sh 'df -h; free -m'
sh 'java -version || true; mvn -v || true; git --version || true'
```

**Linux host checks** (SSH to agent):

```bash
journalctl -u jenkins -xe         # Jenkins service logs
systemctl status jenkins
curl -I https://github.com
nslookup github.com
```

---

## ✅ Memory Tips for 137/OOM

* Increase agent RAM / container limits.
* Reduce parallelism/workers.
* For Java/Maven builds: set heap `-Xmx` smaller or use `MAVEN_OPTS`.

---

## 🧹 Prevent “double checkout”

```groovy
options { skipDefaultCheckout(true) }
```

Then do your single explicit checkout.

---

## 📝 Pro Tips

* Keep pipelines **idempotent** (can re-run safely).
* Prefer **artifact repos** (Nexus/Artifactory) over relying on workspace files.
* Snapshot errors with screenshots; share the **first failing lines**.

---

### 🛟 When stuck

Copy the **first error line** from Console Output and the **few lines above it**, plus your **Jenkinsfile** stage, and ask for help. That’s usually enough to pinpoint the fix quickly. 🙌
