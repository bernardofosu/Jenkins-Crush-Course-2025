## 🤖 Jenkins: Checkout Into Subfolder & Maven Build

### 🛠️ Problem

* Jenkins **does two checkouts**:

  * One implicit checkout → creates `.git` + `dir/` at workspace root.
  * One explicit checkout → into `Boardgame-In--dir-with-jenkins/` subfolder.
* Result: workspace looks like this:

```
$WORKSPACE/
  .git/
  dir/
  Boardgame-In--dir-with-jenkins/
    .git/
    dir/
```

* `@tmp/` folders are just Jenkins’ temporary dirs during checkout.

---

### ✅ Fix — Disable Default Checkout & Clean Workspace

```groovy
pipeline {
  agent any
  options {
    skipDefaultCheckout(true)   // 🚫 stop Jenkins auto-checkout at root
  }

  stages {
    stage('Init (clean)') {
      steps {
        cleanWs()               // 🧹 wipe workspace
      }
    }

    stage('Checkout into subfolder') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[
            url: 'https://github.com/bernardofosu/Boardgame-In--dir-with-jenkins.git',
            credentialsId: 'git-cred'   // 🔑 remove if public
          ]],
          extensions: [[$class: 'RelativeTargetDirectory',
                        relativeTargetDir: 'Boardgame-In--dir-with-jenkins']]
        ])

        // 🔍 sanity checks
        sh 'echo WORKSPACE=$WORKSPACE'
        sh 'ls -la'
        sh 'ls -la Boardgame-In--dir-with-jenkins'
      }
    }
  }
}
```

---

### 📂 Resulting Structure

```
$WORKSPACE/
  Boardgame-In--dir-with-jenkins/
    .git/
    dir/
```

---

### 🚀 Maven Build Options

Because `pom.xml` is **inside**:

```
$WORKSPACE/Boardgame-In--dir-with-jenkins/dir/pom.xml
```

…you must tell Maven where to run:

#### Option 1: Change directory with `dir {}`

```groovy
stage('Build') {
  steps {
    dir('Boardgame-In--dir-with-jenkins/dir') {
      sh 'mvn -B -ntp clean package'
    }
  }
}
```

#### Option 2: Use `-f` to point to `pom.xml`

```groovy
stage('Build') {
  steps {
    sh 'mvn -B -ntp -f Boardgame-In--dir-with-jenkins/dir/pom.xml clean package'
  }
}
```

👉 On Windows agents, replace `sh` with `bat`.

---

### 📦 Archive Artifacts (Optional)

```groovy
archiveArtifacts artifacts: 'Boardgame-In--dir-with-jenkins/dir/target/*.jar', fingerprint: true
```

---

### 💡 Tip

If you remove the explicit “checkout to subfolder” and just clone into the **workspace root**, then you can build directly:

```groovy
sh 'mvn -B -ntp clean package'
```

Because `pom.xml` would then be at:

```
$WORKSPACE/dir/pom.xml
```

✨ Clean checkout = cleaner builds + less confusion!
