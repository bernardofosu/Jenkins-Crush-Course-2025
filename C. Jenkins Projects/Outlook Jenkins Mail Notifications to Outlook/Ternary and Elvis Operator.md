# 🤓 Groovy Operators in Jenkins Pipelines

---

## 1️⃣ Ternary Operator `(condition ? yes : no)`

Classic conditional expression (like in Java, C, Python with inline if/else).

```groovy
def x = 5
def result = (x > 3) ? "yes" : "no"
```

* If `x > 3` → `result = "yes"` ✅
* Else → `result = "no"` ❌

👉 You **must** provide both values (`yes` and `no`).

---

## 2️⃣ Elvis Operator `(value ?: default)`

Shorthand for **“if value is null (or falsey), use default”**.

```groovy
def name = null
def username = name ?: "Guest"
```

* Since `name` is `null`, → `username = "Guest"` 🎉
* If `name = "Bernard"`, → `username = "Bernard"` 🙌

👉 You only provide the **fallback**, not both choices.

---

## 3️⃣ Jenkins Example ⚙️

```groovy
def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
```

* `currentBuild.result` → usually `SUCCESS`, `FAILURE`, or `UNSTABLE`.
* At the very start of a build, it can be `null`.
* The **Elvis operator** ensures:

  * If `currentBuild.result` is set → use it.
  * If `null` → fallback to `'UNKNOWN'`.

So:

* ✅ Run succeeds → `"SUCCESS"`
* ❌ Run fails → `"FAILURE"`
* 🤔 Not yet set → `"UNKNOWN"`

---

## ✅ Key Difference

* **Ternary (`? :`)** → full if-else expression.
* **Elvis (`?:`)** → null-or-default shorthand.

⚡ Think of it like this:

* **Ternary**: “If condition is true, pick A, else pick B.”
* **Elvis**: “If value exists, use it, else use B.”
