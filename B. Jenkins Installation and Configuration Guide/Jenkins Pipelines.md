# 🚀 Jenkins Pipelines — Quick Notes & Examples

## 🧠 What is a Pipeline?

* A **Groovy-based** way to define CI/CD as code.
* Stored in your repo as a **Jenkinsfile** → source-controlled, reviewable, repeatable.

## 🧩 Two Styles

### 1) **Declarative Pipeline** ✅

* **Easier to read & write**
* Opinionated structure (stages/steps)
* Great defaults & validation
* Slightly **limited** for very custom flows

### 2) **Scripted Pipeline** 🧪

* Pure Groovy; **high flexibility**
* More **complex** to maintain
* Best for advanced/conditional logic beyond Declarative’s limits

---

## 🆚 Declarative vs Scripted (at a glance)

| Topic           | Declarative           | Scripted                      |
| --------------- | --------------------- | ----------------------------- |
| Syntax          | `pipeline { ... }`    | `node { ... }` / plain Groovy |
| Learning curve  | Easy                  | Steeper                       |
| Validation      | Strong (editor lints) | Minimal                       |
| Flexibility     | Moderate              | Very High                     |
| Recommended for | Most teams            | Edge cases / heavy logic      |

---

## 📄 Jenkinsfile — **Declarative** example

```groovy
pipeline {
  agent { label 'agent-1' }
  options {
    buildDiscarder(logRotator(numToKeepStr: '3'))
    timestamps()
  }
  environment {
    MAVEN_OPTS = '-Xmx1g'
  }
  tools {
    maven 'Maven-3.9.11'
    jdk 'jdk17'
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/OWNER/REPO.git'
      }
    }
    stage('Build & Test') {
      steps {
        sh 'mvn -B -U clean test'
      }
      post {
        always { junit '*/target/surefire-reports/*.xml' }
      }
    }
    stage('Package') {
      steps { sh 'mvn -B package' }
      post { success { archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true } }
    }
  }
}
```

## 🧾 Jenkinsfile — **Scripted** example

```groovy
node('agent-1') {
  timestamps {
    stage('Checkout') { checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/OWNER/REPO.git']]]) }
    stage('Build & Test') { sh 'mvn -B -U clean test' }
    stage('Package') { sh 'mvn -B package'; archiveArtifacts '**/target/*.jar' }
  }
}
```

---

## 🧰 Pipeline as Code — Workflow

1. Create **Jenkinsfile** in repo root.
2. Push to GitHub/GitLab/Bitbucket.
3. In Jenkins, create a **Multibranch Pipeline** or regular Pipeline pointing to the repo.
4. Add a **webhook** so pushes trigger builds.
5. Jenkins auto-discovers branches/PRs with Jenkinsfiles.

---

## 🏗️ Common Building Blocks

* **agent**: where to run (`any`, `label`, `docker`, etc.)
* **stages/steps**: ordered units of work
* **environment**: key-value vars (can come from credentials)
* **tools**: Maven/JDK/Node versions from **Manage Jenkins → Tools**
* **post**: actions on `success`, `failure`, `always`
* **when**: conditional execution
* **parallel**: run multiple branches concurrently
* **stash/unstash**: pass files between stages/nodes

---

## 🔐 Credentials

* Store in **Manage Jenkins → Credentials**
* Use in pipelines:

```groovy
withCredentials([usernamePassword(credentialsId: 'gh', usernameVariable: 'U', passwordVariable: 'P')]) {
  sh 'git config user.name "$U"'
}
```

---

## ✅ Best Practices

* Prefer **Declarative** for clarity; drop to Scripted only when needed.
* Keep Jenkinsfiles **short**, move complex logic to shared libs.
* Pin tool versions (e.g., `jdk17`, `Maven-3.9.11`).
* Set controller **executors = 0**; run builds on agents.
* Use **log rotation** & **artifact archiving**.
* Add **timestamps()** and **ansiColor('xterm')** for readable logs.

---

## 🧪 Quick Troubleshooting

* Java mismatch? Align **JDK** in Jenkins Tools **and** your `pom.xml`/build tool.
* SCM auth errors? Check **credentials** and webhook.
* Slow builds? Check **CPU credits / EBS IOPS / heap**; use `t3/t4g Unlimited` and `gp3` volumes.

---

> Handy tip: Validate Declarative Jenkinsfiles in **Pipeline Syntax** or **Linter** at `/pipeline-model-converter/validate` on your Jenkins.
