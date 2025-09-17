# 🔗 Upstream vs Downstream (Jenkins) — **Simple Steps**

* **Upstream job** 👉 the job that **triggers** another job.
* **Downstream job** 👈 the job that **gets triggered/started** by the upstream.

We’ll do it two ways:

1. **Freestyle → Freestyle**
2. **Pipeline → Pipeline**

---

## 1️⃣ Freestyle → Freestyle (UI clicks) 🖱️

### Step 1 — Create **upstream-freestyle**

1. **New Item → Freestyle project** → name: **`upstream-freestyle`** → OK.
2. **Build Steps → Execute shell** → put:

   ```bash
   echo "upstream-freestyle"
   ```
3. **Post‑build Actions → Build other projects**.

   * **Projects to build:** `downstream-freestyle`
   * Option: **Trigger only if build is stable** ✅
4. **Save**.

💡 Tip: Separate multiple downstream jobs with commas, e.g., jobB, jobC. A trailing comma isn’t required for a single job (Jenkins usually ignores it if present, but it’s cleaner without it).

### Step 2 — Create **downstream-freestyle**

1. **New Item → Freestyle project** → name: **`downstream-freestyle`** → OK.
2. **Build Steps → Execute shell** → put:

   ```bash
   echo "downstream-freestyle"
   ```
3. **Save**.

### Step 3 — Test

* Click **Build Now** on **upstream-freestyle**.
* Jenkins will **trigger `downstream-freestyle` automatically** after upstream finishes (and is stable).

> ✅ That’s the basic upstream → downstream using pure UI.

---

## 2️⃣ Pipeline → Pipeline (Jenkinsfile) 🧩

### Step 1 — Create **downstream-pipeline**

1. **New Item → Pipeline** → name: **`downstream-pipeline`** → OK.
2. **Pipeline Script** (paste):

   ```groovy
   pipeline {
     agent any

     stages {
       stage('Trigger') {
         steps {
           echo 'Triggered by upstream'
         }
       }
     }
   }
   ```
3. **Save**.

### Step 2 — Create **upstream-pipeline**

1. **New Item → Pipeline** → name: **`upstream-pipeline`** → OK.
2. **Pipeline Script** (paste):

   ```groovy
   pipeline {
     agent any

     stages {
       stage('Hello') {
         steps { echo 'Hello World' }
       }
       stage('Hi') {
         steps { echo 'Hi Ama' }
       }
     }

     post {
       success {
         build job: "downstream-pipeline"   // waits by default
       }
     }
   }
   ```
3. **Save**.

### Step 3 — Test

* Click **Build Now** on **upstream-pipeline**.
* When it **succeeds**, it will trigger **downstream-pipeline** (the upstream waits for it by default).

> 🔎 If your jobs are inside folders, use the full path, e.g., `"team/downstream-pipeline"`.

### Pipeline that fails on a missing tool and then triggers downstream
```sh
pipeline {
  agent any

  stages {
    stage('Hello') {
      steps { echo 'Hello World' }
    }
    stage('Hi') {
      steps { echo 'Hi Ama' }
    }
    stage('Security Scan (will fail)') {
      steps {
        // Intentionally fails if Trivy is NOT installed -> exit code 127
        sh 'trivy -version'
      }
    }
  }

  post {
    // Trigger the downstream ONLY when this upstream failed
    failure {
      echo 'Upstream failed. Triggering downstream anyway...'
      build job: 'downstream-pipeline'  // waits by default
    }
  }
}
```
What happens
- trivy -version isn’t found → stage fails (exit 127) → pipeline result = FAILURE.
- post { failure { … } } runs → downstream-pipeline is triggered even though the upstream failed.

## 📝 Note

**The difference:** Freestyle uses **simple UI clicks** for basic tasks, while **Pipeline uses code (scripts)** which gives you **more flexibility** (conditions, parameters, fan‑out, etc.).

🎉 That’s it—clean, minimal setups for both Freestyle and Pipeline upstream → downstream triggers.

updated the canvas with the exact step-by-step notes you asked for:

Freestyle → Freestyle

Create upstream-freestyle (shell: echo "upstream-freestyle"), add Build other projects → downstream-freestyle, save.

Create downstream-freestyle (shell: echo "downstream-freestyle"), save.

Build upstream-freestyle → triggers downstream-freestyle.

Pipeline → Pipeline

Create downstream-pipeline with your script.

Create upstream-pipeline with your script (triggers downstream-pipeline in post { success { ... } }).

Build upstream-pipeline to see it trigger the downstream.