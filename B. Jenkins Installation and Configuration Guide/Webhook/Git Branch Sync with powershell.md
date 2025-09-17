# 🌿 Git Branch Sync – Step by Step with Emojis

---

## 1️⃣ Confirm Your Remote

```powershell
git remote -v
```

🔍 Should show `origin` pointing to your GitHub repo.

---

## 2️⃣ Fetch All Branches (Not Just Default)

If repo was cloned with **only default branch**, fix the fetch refspec:

```powershell
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
git fetch origin --prune
```

📌 Ensures all remote branches are visible locally.

---

## 3️⃣ List Remote Branches

```powershell
git branch -r
```

📌 You should now see:

* `origin/main`
* `origin/dev`
* `origin/qa`
* `origin/ppd`
* `origin/prod`
* `origin/dr`
* `origin/bugfix`
* `origin/feature`

---

## 4️⃣ Create Local Tracking Branches

Run directly in PowerShell:

```powershell
foreach ($b in "dev","qa","ppd","prod","dr","bugfix","feature") {
  git switch -c $b --track origin/$b
}
```

📌 Creates local branches that track the remotes.

Or save as a script `sync-branches.ps1` and run:

```powershell
.\sync-branches.ps1
```

✅ After running, `git branch` will list them locally.

---

## 5️⃣ Pull Latest for Each Branch (Optional)

```powershell
foreach ($b in "dev","qa","ppd","prod","dr","bugfix","feature") {
  git switch $b
  git pull
}
```

📌 Updates all branches with the latest changes from GitHub.

---

## 6️⃣ Common Gotchas ⚠️

* `git pull` on `main` updates **only main** (not other branches).
* Newly created branches on GitHub need a `git fetch` before they appear.
* Deleted branches? Clean them up:

```powershell
git remote prune origin
```

---

## 🔄 Handy One-Liners

* Fetch & list everything:

```powershell
git fetch --all --prune
git branch -a
```

* Create a single tracking branch:

```powershell
git switch -c dev --track origin/dev
```

---

## 🛠️ Debug if Branches Still Don’t Appear

Run these and check outputs:

```powershell
git remote -v
git branch -a
git config --get remote.origin.fetch
```

📌 These help confirm if the remote fetch setup is correct.

---

✅ With this setup, you can **fetch, track, and sync** all 7 branches easily in PowerShell 🚀
