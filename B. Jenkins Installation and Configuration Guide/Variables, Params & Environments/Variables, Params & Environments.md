# ⚡ Jenkins Pipeline Notes with Variables, Params & Environments

## 🔑 Variable Types

### 1. 🏗️ Built-in Variables

* Provided automatically by Jenkins.
* Examples:

  * `BUILD_NUMBER` → Current build number
  * `BUILD_ID` → Unique build ID
  * `JOB_NAME` → Name of the job
  * `BUILD_URL` → URL to the build
  * `WORKSPACE` → Path to Jenkins workspace
  * `NODE_NAME` → The agent/node label

Usage in script:

```groovy
echo "Build Number: ${env.BUILD_NUMBER}"
echo "Build ID: ${env.BUILD_ID}"
echo "Job Name: ${env.JOB_NAME}"
echo "Workspace: ${env.WORKSPACE}"
echo "Running on Node: ${env.NODE_NAME}"
```

### 🔹 Jenkins Variable Naming
- Environment variables (env.*)
- By convention → UPPERCASE with underscores.
- Examples: BUILD_NUMBER, DEPLOY_ENV, RUN_TESTS.
- Reason: Matches typical shell environment variable style, easier to distinguish from Groovy def variables.

### 2. 🎛️ Parameters

* User-defined inputs when triggering a job.
* Example:

```groovy
parameters {
  string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git branch to build')
}
```

* Access in pipeline:

```groovy
echo "Branch selected: ${params.BRANCH_NAME}"
```

---

### 3. 🌍 Environment Variables

* Define custom environment values in the pipeline.
* Example:

```groovy
environment {
  REGION = 'us-east-1'
}
```

* Access:

```groovy
echo "Deploying to region: ${env.REGION}"
```

---

### 4. 🔹 Using `def`

* `def` is used in Groovy to declare local variables inside pipeline scripts.
* Example:

```groovy
script {
  def version = "1.0.${env.BUILD_NUMBER}"
  echo "App Version: ${version}"
}
```

---

## 🚀 Full Combined Example

```groovy
pipeline {
  agent { label 'agent-1' }

  parameters {
    string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git branch to build')
  }

  environment {
    REGION = 'us-east-1'
  }

  stages {
    stage('Info') {
      steps {
        script {
          echo "Build Number: ${env.BUILD_NUMBER}"
          echo "Build URL: ${env.BUILD_URL}"
          echo "Build ID: ${env.BUILD_ID}"
          echo "Job Name: ${env.JOB_NAME}"
          echo "Workspace: ${env.WORKSPACE}"
          echo "Running on Node: ${env.NODE_NAME}"
          echo "Selected Branch: ${params.BRANCH_NAME}"
          echo "Region: ${env.REGION}"

          def version = "1.0.${env.BUILD_NUMBER}"
          echo "App Version: ${version}"
        }
      }
    }

    stage('Checkout') {
      steps {
        git branch: params.BRANCH_NAME, url: 'https://github.com/bernardofosu/Boardgame-with-Jenkins-MultiPipeline.git'
      }
    }

    stage('Build') {
      steps {
        sh "echo Building app version ${env.BUILD_NUMBER} in region ${env.REGION}"
      }
    }
  }

  post {
    success { echo '✅ Build Succeeded' }
    failure { echo '❌ Build Failed' }
    always  { echo "📅 Job completed at: ${new Date()}" }
  }
}
```

---

## ✅ Summary

* **Built-in variables** → `env.BUILD_NUMBER`, `env.JOB_NAME`, etc.
* **Parameters** → `params.BRANCH_NAME`
* **Environment vars** → `env.REGION`
* **Local vars with def** → `def version = ...`

✨ Combining all makes your Jenkinsfile more dynamic, flexible, and production-ready.

---

# 🧩 Variables in Jenkins (Built‑ins, `params`, `environment`, and `def`)

## 🏷️ Built‑in Environment Variables (auto from Jenkins)

You can access these anywhere with `env.VAR` (inside `script {}`) or `${ENV_VAR}` in shell.

* `BUILD_NUMBER` → Incrementing build number (e.g., `42`)
* `BUILD_ID` → Unique ID timestamped by Jenkins
* `JOB_NAME` → Folder/job path (e.g., `folder/my-job`)
* `BUILD_URL` → Full URL of this build
* `WORKSPACE` → Path to the job’s workspace on the node
* `NODE_NAME` → Name/label of node running the build (e.g., `agent-1`)

🖨️ Example echoes:

```groovy
script {
  echo "#️⃣ BUILD_NUMBER: ${env.BUILD_NUMBER}"
  echo "🧭 JOB_NAME: ${env.JOB_NAME}"
  echo "🔗 BUILD_URL: ${env.BUILD_URL}"
  echo "📂 WORKSPACE: ${env.WORKSPACE}"
  echo "🖥️ NODE_NAME: ${env.NODE_NAME}"
}
```

## 🎛️ `parameters {}` (user inputs at build time)

```groovy
parameters {
  string(name: 'BRANCH_NAME',  defaultValue: 'main',  description: 'Git branch to build')
  booleanParam(name: 'RUN_TESTS', defaultValue: true,  description: 'Run tests?')
  choice(name: 'BUILD_TYPE', choices: ['debug', 'release'], description: 'Build profile')
}
```

Use with `params.BRANCH_NAME`, `params.RUN_TESTS`, etc.

## 🌿 `environment {}` (pipeline-wide key/values)

```groovy
environment {
  REGION = 'xv2'
  OWNER  = 'ci-bot'
}
```

Use with `${env.REGION}` in shell or `env.REGION` in Groovy.

## 🧠 `def` (Groovy variable)

* Declares a **script variable** (scoped to the current `script {}` or pipeline script context).
* Great for **computed** values (e.g., stripping `refs/heads/`).
* Not auto-exported to shell; to use in `sh`, interpolate it into the string.

```groovy
script {
  def shortBranch = (params.BRANCH_NAME ?: 'main').replaceFirst('refs/heads/', '')
  echo "Computed shortBranch = ${shortBranch}"
  sh "echo building for ${shortBranch} in ${env.REGION}"
}
```

> \[!TIP]
> **When to use what?**
> • **Built‑ins** → facts about the build/node.
> • **`parameters`** → operator inputs.
> • **`environment`** → global constants/flags for this run.
> • **`def`** → temporary/computed values in Groovy.

---

# 🧪 Full Combined Pipeline Example (with emojis)

This example uses **label `agent-1`**, shows **parameters**, **environment**, **built‑ins**, **`def`**, and **post actions**.

```groovy
pipeline {
  agent { label 'agent-1' }

  tools {
    maven 'Maven-3.9.11'
    jdk   'jdk17'
  }

  parameters {
    string(name: 'BRANCH_NAME',  defaultValue: 'main', description: 'Git branch to build')
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
    choice(name: 'BUILD_TYPE', choices: ['debug', 'release'], description: 'Build profile')
  }

  environment {
    REGION = 'xv2'
    APP    = 'boardgame'
  }

  stages {
    stage('🌱 Init & Info') {
      steps {
        script {
          // If webhook provided ref=refs/heads/dev, prefer it; fallback to param
          def incomingRef   = binding.hasVariable('ref') ? ref : params.BRANCH_NAME
          def shortBranch   = (incomingRef ?: 'main').replaceFirst('refs/heads/', '')
          // Expose for later stages by putting into env
          env.SHORT_BRANCH  = shortBranch

          echo "🔧 Node: ${env.NODE_NAME} 
#️⃣ Build: ${env.BUILD_NUMBER} 
📦 Job: ${env.JOB_NAME}"
          echo "🌿 Branch (short): ${env.SHORT_BRANCH} | 🌍 Region: ${env.REGION} | 🧩 BuildType: ${params.BUILD_TYPE}"
        }
      }
    }

    stage('📥 Checkout') {
      steps {
        git branch: env.SHORT_BRANCH, url: 'https://github.com/bernardofosu/Boardgame-with-Jenkins-MultiPipeline.git'
      }
    }

    stage('🏗️ Build') {
      steps {
        sh "mvn -B -ntp -P${params.BUILD_TYPE} package"
      }
    }

    stage('🧪 Test (optional)') {
      when { expression { return params.RUN_TESTS } }
      steps {
        sh "mvn -B -ntp test"
      }
    }

    stage('ℹ️ Echo Built-ins') {
      steps {
        script {
          echo "🔗 BUILD_URL: ${env.BUILD_URL}"
          echo "📂 WORKSPACE: ${env.WORKSPACE}"
        }
      }
    }

    stage('🚀 Deploy (demo)') {
      steps {
        sh "echo Deploying ${env.APP}:${env.BUILD_NUMBER} to ${env.REGION} from ${env.SHORT_BRANCH}"
      }
    }
  }

  post {
    success { echo '✅ Pipeline succeeded' }
    failure { echo '❌ Pipeline failed' }
    always  { echo "📅 Finished at: ${new Date()}" }
  }
}
```

---

# 📎 Quick Cheats

* Use **`ref` from webhook**: add Post content param `ref = $.ref`, then compute `short = ref.replaceFirst('refs/heads/','')`.
* **Default `wait`** for `build job:` is `true`; set `wait: false` for async.
* **Images embedded** above for: `post-parameters.png`, `optional-filters.png`, `ref-from.png`, `token.png`.

🎉 You now have a single, production-ready example tying together **webhook params + filters**, **variables**, and **post actions**!
