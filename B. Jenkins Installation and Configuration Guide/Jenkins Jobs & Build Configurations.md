# 📝 Jenkins Jobs & Build Configurations

## 🔹 Jobs / Projects Flow

* Jenkins **Jobs/Projects** → contain tasks.
* **Tasks** are defined in the **build configuration**.
* Build configuration includes:

  * ⚙️ **Compile**
  * 🧪 **Test**
  * 🔍 **Security checks**
  * 🏗️ **Build**
* These steps are then **executed** on agents.

---

## 🔹 Types of Jenkins Jobs

1. **Freestyle Job** 🧩

   * Simple, GUI-driven job.
   * Drag & drop style → easy for small tasks.
   * Example: run a script, build a simple project.

2. **Pipeline Job** 🔗

   * Uses **Jenkinsfile** (Groovy DSL).
   * Supports complex workflows.
   * Best for CI/CD pipelines.

3. **Multibranch Pipeline Job** 🌳

   * Automatically creates pipelines for each branch in a repo.
   * Useful for Git-based workflows (GitHub/GitLab/Bitbucket).
   * Handles PRs and feature branches seamlessly.

---

## 🔹 Freestyle Job Example

* GUI-based, step-by-step.
* Good for **small tasks** or **learning Jenkins**.
* Can run commands, trigger builds, or call external tools.

---

✅ **Summary:**

* Jenkins Jobs define tasks to execute.
* Freestyle → simple.
* Pipeline → advanced automation.
* Multibranch → branch-aware automation.
