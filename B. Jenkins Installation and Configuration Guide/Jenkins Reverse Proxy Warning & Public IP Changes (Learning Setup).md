# 🛠️ Jenkins Reverse Proxy Warning & Public IP Changes (Learning Setup)

## 🧩 Symptoms

```
Manage Jenkins → System diagnostics

It appears that your reverse proxy setup is broken
This message can also appear if you don’t access Jenkins through a reverse proxy:
Make sure the Jenkins URL configured in the System Configuration matches the URL you’re using to access Jenkins.
```

## 🤔 What this really means

Jenkins is generating links using the **Jenkins URL** you set in **Manage Jenkins → System → Jenkins URL**. If your **EC2 public IP changes**, that URL becomes wrong. Then the browser tries to load CSS/JS/WebSocket from the **old address**, causing timeouts and a very **slow UI**.

> 🔴 **Very important:** Update the **Jenkins URL** whenever your instance’s **public IP** changes to avoid sluggish behavior and broken links.

---

## ✅ Quick Fix (Every time the public IP changes)

1. **Find the current public IP** of the controller:

   * From the controller host:

     ```bash
     curl -s http://checkip.amazonaws.com
     ```
   * Or in AWS Console → **EC2 → Instances → Public IPv4 address**
2. Open **Jenkins** in your browser (using that new IP):

   * `http://<NEW_PUBLIC_IP>:8080/`
     *(Add `/jenkins/` if you run behind a context path)*
3. Go to **Manage Jenkins → System**.
4. In **Jenkins URL**, paste the exact URL you use in the browser, e.g.:

   * `http://<NEW_PUBLIC_IP>:8080/`
   * or `https://<your-domain>/jenkins/` (if you use a reverse proxy + TLS)
5. **Save**.
6. Reload the **Manage Jenkins** page. The warning should disappear.

> 💡 If you’re **not** using a reverse proxy at all, simply set the Jenkins URL to the direct address you actually visit (usually `http://<PUBLIC_IP>:8080/`).

---

## 🧭 Context Path (if you proxy under /jenkins)

* If you access Jenkins at `https://example.com/jenkins/`, make sure **both** are aligned:

  * Jenkins runs with `--prefix=/jenkins`
  * **Jenkins URL** is `https://example.com/jenkins/` (note the trailing slash)

---

## 🧪 Sanity Checks

* **From a client:**

  ```bash
  curl -I http://<NEW_PUBLIC_IP>:8080/login
  # Expect 200 OK (or 403 if unauthorized), but NOT a redirect to an old IP/URL
  ```
* **In the browser devtools → Network tab:** ensure CSS/JS requests point to the **new** IP/URL, not the old one.

---

## ⚡ Optional: Update via Script Console (Advanced)

If you prefer to change the URL programmatically (useful for quick updates during learning):

1. Open **Manage Jenkins → Script Console**
2. Run:

```groovy
import jenkins.model.JenkinsLocationConfiguration

def jlc = JenkinsLocationConfiguration.get()
jlc.setUrl("http://<NEW_PUBLIC_IP>:8080/")
jlc.save()
println "Jenkins URL updated to: ${jlc.getUrl()}"
```

*(Replace with your exact URL. If you use a context path, include the trailing slash, e.g., `https://example.com/jenkins/`.)*

---

## 📌 Very Important Notes

* 🧭 **Always update the Jenkins URL after a public IP change.**
* 🐢 **Slow UI = stale URL** (browser fetching assets from an old address → timeouts).
* 🔒 If you use a reverse proxy (Nginx/ALB/Apache), ensure it sets `X-Forwarded-Proto`, `X-Forwarded-Port`, and preserves the `Host` header.
* 🧩 Keep the **context path** consistent (`--prefix=/jenkins` if you proxy at `/jenkins`).

---

## 🧯 Troubleshooting Quick List

* ✅ **Jenkins URL** matches what you type in the browser (scheme/host/port/path)
* ✅ No redirects to old hosts/ports in `curl -I` responses
* ✅ Proxy headers present (if using a reverse proxy):

  * `X-Forwarded-Proto` = `https` when TLS terminates at the proxy
  * `X-Forwarded-Port` matches external port (443/80)
  * `Host` header is preserved
* ✅ If using agents, their SSH settings can use **private IPs** (independent of Jenkins URL)

---

## 🚀 Nice-to-have (when you move beyond learning)

* 📎 **Elastic IP** for the controller → the IP doesn’t change
* 🌐 **DNS (Route 53)** pointing a domain (e.g., `ci.example.com`) to the EIP
* 🔐 HTTPS via Nginx/ALB + Let’s Encrypt, and set **Jenkins URL** to `https://ci.example.com/`

---

## 📝 TL;DR

> **If your EC2 public IP changes, immediately update `Manage Jenkins → System → Jenkins URL` to the new address.**
> This prevents slow loading and removes the “reverse proxy setup is broken” warning. 🟢

---

## ⚙️ Maven in Pipeline: `mvn: not found` (SSH Agent)

### 🧯 Error seen in Console

```
+ mvn compile
/home/ubuntu/jenkins/workspace/First-Pipeline@tmp/durable-325398d6/script.sh.copy: 1: mvn: not found
```

### 💡 Why this happens

Even if you configure **Maven** under **Manage Jenkins → Global Tool Configuration**, the **Pipeline does not automatically get Maven on PATH**. You **must** reference the tool in the Pipeline (or install Maven on the agent and ensure PATH). Otherwise, any `sh 'mvn ...'` step fails with `mvn: not found`.

> 🔴 **Important:** Defining Maven in *Global Tool Configuration* is not enough. Add a **`tools { ... }`** block (Declarative) or use **`tool` / `withMaven`** (Scripted) so the build knows which Maven to use.

---

### ✅ Fix Option A — Declarative Pipeline with `tools {}` (recommended)

```groovy
pipeline {
  agent any
  tools {
    jdk   'JDK17'        // Name exactly as defined in Global Tool Configuration
    maven 'Maven-3.9'    // Ditto; enable "Install automatically" for agents
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn -v'                     // prove Maven is on PATH now
        sh 'mvn -B -ntp clean test'     // your build
      }
    }
  }
}
```

### ✅ Fix Option B — Scripted: `tool` + `withEnv`

```groovy
def mvnHome = tool name: 'Maven-3.9', type: 'maven'
withEnv(["PATH+MAVEN=${mvnHome}/bin"]) {
  sh 'mvn -v'
  sh 'mvn -B -ntp clean verify'
}
```

### ✅ Fix Option C — Using `withMaven` (Pipeline Maven Integration plugin)

```groovy
withMaven(maven: 'Maven-3.9', jdk: 'JDK17') {
  sh 'mvn -B -ntp clean verify'
}
```

---

### 🧪 Quick Checklist

* 🔑 **Global Tool Configuration** has **JDK** and **Maven** defined (names match your Pipeline).
* ⚙️ **Pipeline** uses **`tools { … }`** or **`tool`/`withMaven`** to put Maven on PATH.
* 🖥️ If you prefer system Maven, install it on the **agent** and verify PATH:

  ```bash
  sudo apt update && sudo apt install -y maven
  mvn -v
  ```

  *(Using Jenkins tools is recommended for consistent, auto-managed versions.)*

### 📝 TL;DR

> Add a **`tools` block** (or `withMaven`/`tool`) — otherwise your Pipeline won’t see Maven and you’ll get **`mvn: not found`**. 🚀

---

# ⚙️ Jenkins + Maven Pipeline Errors (Learning Notes)

## ❗ Error Snippet #1 — Maven not found

```
+ mvn compile
/home/ubuntu/jenkins/workspace/First-Pipeline@tmp/durable-325398d6/script.sh.copy: 1: mvn: not found
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test)
```

### 💡 Why it happens

Even if you define **Maven** under **Manage Jenkins → Global Tool Configuration**, the **Pipeline will NOT automatically know about it**. You must expose the tool to the build with a **`tools { ... }`** block (Declarative) or via **`tool`/`withMaven`** (Scripted). Otherwise `mvn` isn’t on `PATH` during the build.

### ✅ Quick Fix (Declarative)

```groovy
pipeline {
  agent any
  tools {
    jdk   'JDK17'      // name must match Global Tool Configuration
    maven 'Maven-3.9'  // enable "Install automatically" for agents
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn -v'               // sanity check
        sh 'mvn -B -ntp clean test'
      }
    }
  }
}
```

### ✅ Scripted alternatives

```groovy
// A) tool + PATH
 def mvnHome = tool name: 'Maven-3.9', type: 'maven'
 withEnv(["PATH+MAVEN=${mvnHome}/bin"]) {
   sh 'mvn -v'
   sh 'mvn -B -ntp clean verify'
 }

// B) withMaven (Pipeline Maven Integration plugin)
 withMaven(maven: 'Maven-3.9', jdk: 'JDK17') {
   sh 'mvn -B -ntp clean verify'
 }
```

> 📝 **Key reminder:** Configuring tools in Jenkins is not enough—**reference them in the Pipeline**.

---

## ❗ Error Snippet #2 — Compiler crash on newer JDK

```
[INFO] Total time:  2.861 s
[INFO] Finished at: 2025-09-10T22:26:06Z
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.1:compile (default-compile) on project database_service_project: Fatal error compiling: java.lang.NoSuchFieldError: Class com.sun.tools.javac.tree.JCTree$JCImport does not have member field 'com.sun.tools.javac.tree.JCTree qualid' -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
```

### 💡 What it usually means

This typically indicates a **JDK / annotation-processor / plugin mismatch**. Common causes:

* Building with **JDK 21** (or newer) while your build uses tools or processors compiled against older `javac` internals (e.g., Error Prone, outdated annotation processors, or plugins that touch `com.sun.tools.javac.*`).
* Old **maven-compiler-plugin (3.8.1)** with newer JDK features.

### ✅ Fixes (pick one or combine)

1. **Use JDK 17 LTS for the build** (often the quickest path):

   * In Pipeline add:

     ```groovy
     tools { jdk 'JDK17'; maven 'Maven-3.9' }
     ```
   * Or set a **Maven Toolchain** to force JDK 17.

2. **Update the Maven Compiler Plugin** and set a stable release level:

   ```xml
   <plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-compiler-plugin</artifactId>
     <version>3.11.0</version>
     <configuration>
       <release>17</release> <!-- or 11/8 to match your codebase -->
     </configuration>
   </plugin>
   ```

3. **Upgrade/align annotation processors** (e.g., Lombok, MapStruct, Error Prone):

   * Use versions compatible with your chosen JDK.
   * If using **Error Prone**, upgrade it; or temporarily **disable it** on CI while you stabilize JDK.

4. **Ensure a real JDK (not just a JRE)** is used to compile:

   * On agents, verify `javac -version` works, or rely on Jenkins **JDK tool**.

### 🔍 Debug tips

* Print which Java/Maven are used:

  ```groovy
  sh 'java -version && javac -version && mvn -v'
  ```
* Try a different JDK quickly by changing the **tools** block (e.g., JDK17).
* Re-run with more logs: `mvn -e -X -B -ntp clean compile`.

---

## 🧭 TL;DR

* 🧰 **Always expose Maven/JDK to the Pipeline** via `tools { ... }` or `withMaven` → fixes `mvn: not found`.
* 🏗️ **Target a compatible JDK (often 17 LTS)** and **update maven-compiler-plugin** (e.g., 3.11.0) → fixes the `NoSuchFieldError` from `com.sun.tools.javac.*` mismatches.
* 🔄 Keep annotation processors/plugins up to date.
