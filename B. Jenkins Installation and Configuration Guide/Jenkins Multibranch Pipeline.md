# 🌿 Jenkins Multibranch Pipeline – Notes with Emojis

---

## 🏗️ Repository & Branch Setup

* GitHub repo connected to Jenkins ✅
* **Branches created:**

  * `dev`
  * `qa`
  * `ppd`
  * `prod`
  * `dr`
  * `bugfix`
  * `feature`

---

## 📂 Jenkinsfile Status

* **Jenkinsfile Present:** `dev`, `qa`, `ppd`, `prod`, `dr`

  * ✅ Jenkins auto-detects pipelines for these branches.
* **Jenkinsfile Deleted:** `bugfix`, `feature`

  * ❌ Jenkins skips these branches (status: *No Jenkinsfile*).

---

## 🔄 How Multibranch Pipeline Works

1. Jenkins scans the GitHub repo.
2. For each branch → checks if a `Jenkinsfile` exists.
3. ✅ If found → creates a pipeline job automatically.
4. ❌ If missing → branch is shown but build is disabled.

---

## ⚡ Benefits

* 🔎 **Automatic branch discovery** – no manual job creation.
* 🧩 **Environment isolation** – each branch can define its own CI/CD flow.
* 📦 **Consistent pipelines** – Dev → QA → Prod all follow the same structure.
* 🛡️ **Flexibility** – selectively enable/disable branches with `Jenkinsfile` presence.

---

## 📝 Key Observations

* 7 branches in total.
* **Active pipelines:** 5 branches (Dev, QA, PPD, Prod, DR).
* **Skipped pipelines:** 2 branches (Bugfix, Feature).
* Jenkins log during scan will highlight missing Jenkinsfiles.

---

# ✅ Summary

* Multibranch pipeline job created in Jenkins.
* Repo scanned → Jenkinsfile found in **5 branches**, jobs created.
* **Bugfix & Feature skipped intentionally** (no Jenkinsfile).
* Multibranch = efficient CI/CD across multiple environments 🚀.
