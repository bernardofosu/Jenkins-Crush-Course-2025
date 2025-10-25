## 🧹 Jenkins Workspace Cleanup: `cleanWs()` vs `deleteDir()`

### 🔄 What `cleanWs()` Does

```groovy
post {
  always {
    echo 'Cleaning up workspace...'
    cleanWs()
  }
}
```

* ✅ **Wipes the job’s workspace** after stages finish (success, failure, or aborted).
* ✅ Deletes **everything under `$WORKSPACE`**, including hidden files and Jenkins’ temp dirs (`@tmp`).
* 🚫 Does **not** delete archived artifacts, logs, or files outside the workspace.

**Why use it?**

* 🗑️ Free disk space
* 🧼 Avoid leftover files interfering with next builds
* 🔄 Force a fresh checkout next time

---

### ⚙️ Common Options (Workspace Cleanup Plugin)

```groovy
cleanWs(
  deleteDirs: true,           // 🗂️ also remove empty dirs
  notFailBuild: true,         // 🚫 don’t fail build if cleanup hits a locked file
  disableDeferredWipeout: true, // ⚡ immediate wipeout instead of delayed
  patterns: [
    [pattern: '**/*.log', type: 'INCLUDE'],  // 📝 only clean logs
    [pattern: '**/keep.me', type: 'EXCLUDE'] // 🙅 keep specific files
  ]
)
```

---

### ❌ `deleteDir()` (built-in)

* Clears the current dir, but **no patterns/excludes**.
* Less safe → issues with Windows long paths or locked files.
* Simpler but less flexible.

---

### ✨ Best Practices

* ✅ Use `cleanWs()` (safer + flexible).
* 🔍 Want to keep workspace for debugging?

```groovy
post {
  success { cleanWs() }   // only clean on success
}
```

Or:

```groovy
cleanWs(skipWhenFailed: true)
```

(if your plugin version supports it).

---

### 📝 TL;DR

* `cleanWs()` → **recommended** (Workspace Cleanup plugin).
* `deleteDir()` → **basic** cleanup, no filters.
* Use cleanup wisely to keep builds fresh, fast, and reliable 🚀.
