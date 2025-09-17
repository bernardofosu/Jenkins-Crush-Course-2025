## 🧩 Generic Webhook Trigger — Post Content Parameters & Filters

> These options apply when you use the **Generic Webhook Trigger** plugin (single Pipeline/Freestyle jobs). Multibranch jobs usually don’t need this.

### 1) Post content parameters (extract values from JSON)

**What it does:** Parses the incoming webhook JSON and writes values to **environment variables** you can use in your job.

**Example (GitHub push):** GitHub sends a field `ref` with the branch or tag, e.g. `"refs/heads/dev"`.

* **Variable:** `ref`
* **Expression:** `$.ref`  *(JSONPath to that field)*

**How to use it in a Pipeline:**

```groovy
// Groovy echo
echo "ref is ${env.ref}"

// Shell step
sh 'echo ref=$ref'
```

> Tip: For PR events, the branch is at a different path, e.g. `$.pull_request.head.ref`.

### 2) Optional Filters 🎯 (decide whether to trigger the build)

**What it does:** Runs a **regex match** and only triggers the job when it matches.

**Fields:**

* **Expression** → a regex pattern (e.g., `^refs/heads/dev$`)
* **Text** → where to read the value from (usually the variable you just extracted), e.g. `$ref`

**Example: trigger only on `dev` branch**

* **Expression:** `refs/heads/dev$`
* **Text:** `$ref`

**More examples:**

* Only `main` or `dev`:
  Expression: `^refs/heads/(main|dev)$`  ·  Text: `$ref`
* Only tags:
  Expression: `^refs/tags/.*`  ·  Text: `$ref`
* Only PRs from `feature/*` (for PR payloads):
  Expression: `^feature/.+$`  ·  Text: `$pull_request_head_ref` *(extract it first with JSONPath `$.pull_request.head.ref` and name it `pull_request_head_ref`)*

**What happens at runtime:**

1. Plugin extracts variables via your **Post content parameters**.
2. It evaluates **Filters**. If the regex **matches**, the job triggers; if not, it **skips** (check the job console to see the reason).

**Using `ref` to check out the branch (non‑multibranch):**

```groovy
// Strip the prefix refs/heads/ to get the plain branch name
def branch = (env.ref ?: '').replace('refs/heads/','')
checkout([$class: 'GitSCM', branches: [[name: branch]], userRemoteConfigs: [[url: 'https://github.com/owner/repo.git']]])
```

**Debug tips:** enable “Print post content” in the trigger, and try a test delivery from GitHub to see the payload and your extracted values.
