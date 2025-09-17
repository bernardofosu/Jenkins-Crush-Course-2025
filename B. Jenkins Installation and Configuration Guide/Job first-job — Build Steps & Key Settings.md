# 🧱 Job: **first-job** — Build Steps & Key Settings

## 🔧 General

* **Description:** `first build`
* **Build retention:** ✅ *Discard old builds* → **Log Rotation**

  * **Max # of builds to keep:** `3`
  * *(Days empty → only the last 3 builds are kept)*
* **This project is parameterized:** ✅

  * **String Parameter**

    * **Name:** `Staging Code`
    * **Default value:** `staging`
    * **Description:** `Staging Code`
* **Restrict where this project can be run:** ✅

  * **Label Expression:** `agent-1` *(runs only on that agent)*

## 🧭 Source Code Management (SCM)

* **Type:** Git
* **Repository URL:** [https://github.com/bernardofosu/Boardgame-with-jenkins.git](https://github.com/bernardofosu/Boardgame-with-jenkins.git)
* **Credentials:** `- none -` *(public repo)*
* **Branch Specifier:** `main`

## ⏱ Triggers

* **None** enabled yet *(no Poll SCM / webhook). You’ll run manually or add a trigger later.*

## 🌿 Build Environment

* No boxes ticked.

  * *Nice to consider later:* **Add timestamps to the Console Output** and **Delete workspace before build starts** for clean builds.

## 🧪 Build Steps (Maven)

You defined three Maven steps, each using **Maven-3.9.11**:

1. **Compile**
   **Goals:** `compile`
2. **Test**
   **Goals:** `test`
3. **Package**
   **Goals:** `package`

> This effectively runs the usual lifecycle in separate steps: **compile → test → package** (produces the artifact, e.g., JAR/WAR).

## 📦 Post-Build Actions

* **None** *(you can add **Archive the artifacts** later to keep your JAR/WAR in Jenkins, or Publish JUnit results).*

## ✅ Pre-req checks (quick sanity)

* **Agent has Java:** `java -version` *(needed for Jenkins agent & Maven).*
* **Maven tool configured:** *Manage Jenkins → Tools → Maven installations* has **Maven-3.9.11** *(already used by the job).*

## 💡 Nice improvements (optional)

* **Archive artifacts:** *Post-build →* **Archive the artifacts** → `**/target/*.jar`
* **Timestamps:** *Environment →* **Add timestamps to the Console Output**
* **SCM trigger:** **GitHub hook trigger for GITScm polling** *(with GitHub webhook)* **or** **Poll SCM** (`H/2 * * * *`) for every \~2 min
* **Workspace cleanup:** **Delete workspace before build starts** if you see caching conflicts
* **Controller executors:** Keep controller at **0** executors; builds only on `agent-1`

## 🔁 (Bonus) Pipeline equivalent (if you later convert to Jenkinsfile)

```groovy
pipeline {
  agent { label 'agent-1' }
  options { buildDiscarder(logRotator(numToKeepStr: '3')) }
  parameters {
    string(name: 'Staging Code', defaultValue: 'staging', description: 'Staging Code')
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/bernardofosu/Boardgame-with-jenkins.git'
      }
    }
    stage('Compile') { steps { sh 'mvn -B compile' } }
    stage('Test')    { steps { sh 'mvn -B test' } }
    stage('Package') { steps { sh 'mvn -B package' } }
  }
}
```

---

## 🔧 JDK / Maven Troubleshooting — **Java Version Mismatch**

### 📌 What we saw

* Project **POM** declares **Java 11**:

```xml
<properties>
  <java.version>11</java.version>
  <jacoco.version>0.8.7</jacoco.version>
  <sonar.java.coveragePlugin>jacoco</sonar.java.coveragePlugin>
  <sonar.dynamicAnalysis>reuseReports</sonar.dynamicAnalysis>
  <sonar.jacoco.reportPath>${project.basedir}/../target/jacoco.exec</sonar.jacoco.reportPath>
  <sonar.language>java</sonar.language>
</properties>
```

* Jenkins controller/agent initially used **JDK 21**, causing `javac`/plugin incompatibility.
* Build error from Maven:

```text
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.1:compile (default-compile)
[ERROR] Fatal error compiling: java.lang.NoSuchFieldError: Class com.sun.tools.javac.tree.JCTree$JCImport does not have member field 'com.sun.tools.javac.tree.JCTree qualid'
[ERROR] Re-run with -e or -X for full stack trace. See: http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
Build step 'Invoke top-level Maven targets' marked build as failure
Finished: FAILURE
```

### ✅ Fix we applied

1. **Install multiple JDKs** using the **Eclipse Temurin installer** plugin:

   * *Manage Jenkins → Plugins → Available* → **Eclipse Temurin installer** → Install & restart.
   * *Manage Jenkins → Tools → JDK installations* → Add:

     * `jdk21` (Install automatically → adoptium **jdk-21.0.x**)
     * `jdk17` (Install automatically → adoptium **jdk-17.0.x**)
2. **Select JDK per job**:

   * In job **Configure → JDK to be used for this project** → **`jdk17`** (matches tool you added).
   * Job remains labeled to run on **agent-1**.

> Using **JDK 17** avoids the `JCTree` error while keeping a modern runtime. (If the project truly requires 11, add `jdk11` and select it for the job.)

### 🛠️ Alternative (keep Java 21): update POM & plugins

If you prefer Java 21, bump toolchain & plugins in `pom.xml`:

```xml
<properties>
  <java.version>21</java.version>
  <maven.compiler.release>21</maven.compiler.release>
  <jacoco.version>0.8.11</jacoco.version>
</properties>

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.11.0</version>
      <configuration>
        <release>${maven.compiler.release}</release>
      </configuration>
    </plugin>
    <!-- Consider updating surefire/failsafe if tests fail on new JDK -->
  </plugins>
</build>
```

### 📁 Where the artifact is

* After **package**, the output is in the job **Workspace** under `target/`
  *(e.g., `…/workspace/first-job/target/*.jar`)*
  You can also add **Post-build → Archive the artifacts** with pattern `**/target/*.jar` to keep it in Jenkins.

### 🔎 Quick checklist

* [x] JDKs installed via Temurin plugin (`jdk17`, `jdk21`).
* [x] Job set to use **jdk17**.
* [x] Build steps: `compile → test → package` (Maven 3.9.11).
* [x] Artifact verified in **Workspace → target/**.
