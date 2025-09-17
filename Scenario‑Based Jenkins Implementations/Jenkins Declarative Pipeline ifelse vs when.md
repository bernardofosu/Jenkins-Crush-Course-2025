## 🤖 Jenkins Declarative Pipeline: `if/else` vs `when {}`

### 🚫 Why You Can’t Use Bare `if/else` in `steps {}`

* `steps {}` in Declarative pipelines only allows Jenkins steps (`sh`, `echo`, `archiveArtifacts`, …).
* `if/else`, loops, and variables are **Groovy code**, not Jenkins steps.
* To use them, you must switch into "scripted" mode with `script { ... }`.

---

### ✅ Option A — Use `script {}` for Groovy logic

```groovy
pipeline {
  agent any

  parameters {
    choice(name: 'ENV', choices: ['Dev', 'QA', 'Prod'], description: 'environment')
  }

  stages {
    stage('Deploy') {
      steps {
        script {
          if (params.ENV == 'Dev') {
            sh 'echo dev'
          } else if (params.ENV == 'QA') {
            sh 'echo QA'
          } else {
            sh 'echo Prod'
          }
        }
      }
    }
  }
}
```

👉 Wrap logic in `script {}` to enable Groovy `if/else` inside Declarative.

---

### ✅ Option B — Pure Declarative with `when {}`

```groovy
pipeline {
  agent any

  parameters {
    choice(name: 'ENV', choices: ['Dev', 'QA', 'Prod'], description: 'environment')
  }

  stages {
    stage('Deploy to Dev') {
      when { expression { params.ENV == 'Dev' } }
      steps { sh 'echo dev' }
    }

    stage('Deploy to QA') {
      when { expression { params.ENV == 'QA' } }
      steps { sh 'echo QA' }
    }

    stage('Deploy to Prod') {
      when { expression { params.ENV == 'Prod' } }
      steps { sh 'echo Prod' }
    }
  }
}
```

👉 Each stage only runs if its `when` condition is true.

---

### 📌 `def` vs `script {}`

* **`def`** → Groovy keyword for declaring variables/functions (`def x = 1`).
* **`script {}`** → Jenkins step that unlocks Groovy execution inside Declarative pipelines.

---

### ⚡ TL;DR

* Use `script { if (...) { ... } }` for inline control flow.
* Use `when { expression { ... } }` for cleaner **Declarative-only** conditions.

✨ This way, you balance flexibility (Groovy) with the structure of Declarative pipelines!
