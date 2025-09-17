# 🔒 Restricting the `main` Branch on GitHub

You can restrict the **main** branch on GitHub, even in your **personal
repositories** --- it's not only for organizations.

------------------------------------------------------------------------

## ✅ How to Restrict `main`

1.  Go to your repo on GitHub.\
2.  Click **Settings → Branches → Branch protection rules**.\
3.  Add a rule for `main`.\
4.  Options you can enable:
    -   ✅ Require pull requests before merging.\
    -   ✅ Require status checks to pass before merging.\
    -   ✅ Require approvals (e.g., 1--2 reviewers).\
    -   ✅ Prevent force pushes.\
    -   ✅ Prevent deletions.\
    -   ✅ Restrict who can push to `main`.

------------------------------------------------------------------------

## ⚡ Examples

-   **Personal repo (solo dev)** → tick *"Require pull request"* so you
    don't accidentally push directly to `main`.\
-   **Team repo (org)** → add *required reviews* + *status checks* + *no
    force push*.

------------------------------------------------------------------------

## 🔑 TL;DR

-   Yes, you can restrict `main` in your **personal repos**.\
-   Organization repos give **more control** (like requiring reviews
    from specific teams).

------------------------------------------------------------------------

👉 Do you want me to show you a **step-by-step screenshot-style flow**
so you can follow along when setting the protection?
