# 🌿 Environment Block

### Declares pipeline-wide constants

```groovy
environment {
  REGION = 'xv2'
  OWNER  = 'ci-bot'
}
```

📌 Access with `env.REGION` or `${REGION}`.

---

# 🧠 def Variables

### Local Groovy variables (not exported)

```groovy
script {
  def shortBranch = ref.replaceFirst('refs/heads/', '')
  echo "Branch = ${shortBranch}"
}
```

👉 If needed across stages, assign to env:

```groovy
env.SHORT_BRANCH = shortBranch
```

---

# ⚡ Runtime Environment Variables

### Create or update variables dynamically during a pipeline run

```groovy
stage('Dynamic Custom Vars') {
  steps {
    script {
      env.RUNTIME_VAR = "runtime-${env.BUILD_NUMBER}"
      echo "⚡ DYNAMIC: ${env.RUNTIME_VAR}"
    }
  }
}
```

✅ Available in later stages just like normal env vars.

---

# ✅ Summary

* **Post content parameter** = values extracted from webhook payload.
* **Optional filter** = controls whether job starts.
* **Token** = secure trigger.
* **Built-in env** = provided automatically.
* **parameters** = user-specified at build time.
* **environment {}** = global constants for pipeline.
* **def** = temporary Groovy vars (local scope).
* **env.RUNTIME\_VAR** = dynamic runtime variable you can set inside pipeline.

✨ This gives you full flexibility: constants, parameters, built-ins, computed values, and runtime-generated env vars all in one pipeline!
