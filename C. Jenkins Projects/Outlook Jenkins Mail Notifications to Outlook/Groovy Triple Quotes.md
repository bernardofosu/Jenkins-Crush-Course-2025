# 📘 Groovy Triple Quotes (`""" ... """`)

---

## 🔹 What `"""` Means

In Groovy, strings can be written in different ways:

* `"Hello"` → normal string.
* `'Hello'` → single-quoted string.
* `"""Hello"""` → triple-double-quoted string.
* `'''Hello'''` → triple-single-quoted string.

---

## 🔹 Why Use Triple Quotes?

### ✅ Multiline Support

You can spread the string over multiple lines without `+` concatenation.

```groovy
def text = """
Line 1
Line 2
Line 3
"""
```

Output:

```
Line 1
Line 2
Line 3
```

---

### ✅ String Interpolation

Triple-double-quoted (`"""`) strings support `${variables}` inside them.

```groovy
def name = "Bernard"
def msg = """Hello ${name}, welcome!"""
```

➡️ Output: `Hello Bernard, welcome!`

---

### ✅ Clean HTML/JSON Blocks

Perfect for embedding **HTML, JSON, YAML, or shell scripts** in Jenkins pipelines without breaking formatting.

---

## 🔹 In Your Case

```groovy
def body = """
<html>
<body>
<div style="border: 4px solid ${bannerColor}; padding: 10px;">
<h2>${jobName} - Build ${buildNumber}</h2>
<div style="background-color: ${bannerColor}; padding: 10px;">
<h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
</div>
<p>Check the <a href="${env.BUILD_URL}">console output</a>.</p>
</div>
</body>
</html>
"""
```

* The **HTML spans multiple lines**, so triple quotes keep it readable.
* Inside, `${jobName}`, `${buildNumber}`, `${pipelineStatus}`, and `${env.BUILD_URL}` get interpolated at runtime.

✅ So yes → `"""` is used here because of **multiline support** and **string interpolation**.

---

## 🔹 Multiline Shell Script Example

It’s the same idea when writing **multiline shell scripts** in Jenkins pipelines:

```groovy
pipeline {
    agent any
    stages {
        stage('Script Example') {
            steps {
                sh """
                  echo "Hello from line 1"
                  echo "Hello from line 2"
                  echo "Hello from line 3"
                """
            }
        }
    }
}
```

* `""" ... """` lets you write **multiple shell commands** clearly.
* Without it:

  ```groovy
  sh "echo Hello from line 1; echo Hello from line 2; echo Hello from line 3"
  ```

  👎 Much harder to read.

---

## ✅ Summary

* **Groovy Triple Quotes = Multiline + Interpolation**.
* Perfect for **HTML, JSON, or scripts** in Jenkins pipelines.
* Works the same way as writing **multiline shell commands** with `sh """ ... """`.

⚡ Makes your Jenkinsfiles **cleaner, readable, and more powerful**!
