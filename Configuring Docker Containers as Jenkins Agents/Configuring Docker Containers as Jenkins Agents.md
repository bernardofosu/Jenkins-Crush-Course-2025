# 🐳 Configuring Docker Containers as Jenkins Agents

---

## 🔹 **1. Prerequisites**

* 🖥️ Jenkins Master (installed or running in a container).
* 🐳 Docker Daemon installed and running on Jenkins master host.
* 👤 Jenkins user added to the Docker group (to access Docker socket).

### 📜 Commands (Ubuntu)

```bash
# Install Docker
sudo apt-get update
sudo apt-get install -y docker.io

# Add Jenkins user to Docker group
sudo usermod -aG docker jenkins

# Restart Jenkins and Docker
sudo systemctl restart docker
sudo systemctl restart jenkins

# Verify Docker access
docker info
```

---

## 🔹 **2. Install Jenkins Plugins**

1. Go to **Manage Jenkins → Manage Plugins → Available**.
2. Install:

   * 📦 Docker plugin
   * 📦 Docker Pipeline plugin
3. 🔄 Restart Jenkins if required.


## 🔹 **3. Configure Docker Cloud in Jenkins**

1️⃣ Navigate to:
👉 **Manage Jenkins → Manage Nodes and Clouds → Configure Clouds → Add a new cloud → Docker**

2️⃣ **Docker Cloud Details:**

| 📌 Field               | 🔧 Value                       |
| ---------------------- | ------------------------------ |
| 🌐 Name                | docker-cloud                   |
| 🐳 Docker Host URI     | `unix:///var/run/docker.sock`  |
| 🔑 Server Credentials  | Leave blank (local Docker)     |
| ✅ Enabled              | ✔️                             |
| 📦 Container Cap       | 100 (adjust as needed)         |
| 🚫 Expose DOCKER\_HOST | Unchecked (unless DinD needed) |

➡️ Click **Test Connection** → It should display Docker API version.

---

## 🔹 **4. Configure Docker Agent Template**

1️⃣ Inside Docker Cloud config → **Add Docker Template**

2️⃣ **Template Settings:**

| 📌 Field              | 🔧 Value                                    |
| --------------------- | ------------------------------------------- |
| 🏷️ Labels            | docker-agent                                |
| 🐳 Docker Image       | `jenkins/inbound-agent:latest` (or custom)  |
| 📂 Remote FS Root     | `/home/jenkins`                             |
| 🔗 Connect Method     | Attach Docker container                     |
| 👤 User               | Leave blank (defaults to image user)        |
| 📥 Pull Strategy      | Pull once and update                        |
| ♻️ Retention Strategy | Remove after use                            |
| 📦 Volumes (Optional) | `/var/run/docker.sock:/var/run/docker.sock` |


## 🔹 **5. Create a Custom Docker Agent (Optional)**

📜 **Dockerfile Example:**

```dockerfile
FROM jenkins/inbound-agent:latest

# Switch to root user to install packages
USER root

# Install Java 17 (headless) and Maven
RUN apt-get update && \
    apt-get install -y openjdk-17-jre-headless maven && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set JAVA_HOME environment variable
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
ENV PATH=$JAVA_HOME/bin:$PATH

# Switch back to Jenkins user
USER jenkins
```

### 🏗️ Build Docker Image

```bash
docker build -t your-username/imagename:latest .
```

### 📤 Push Docker Image

```bash
docker login
docker push your-username/imagename:latest
```

---

## 🔹 **6. Test the Docker Agent with a Pipeline**

📜 **Jenkinsfile Example:**

```groovy
pipeline {
    agent { label 'docker-agent' }
    stages {
        stage('Verify Agent') {
            steps {
                sh 'echo "Hello from Docker Agent!"'
                sh 'whoami'
                sh 'uname -a'
            }
        }
    }
}
```

✅ Run the pipeline → Jenkins will spin up a Docker agent and execute the steps inside the container 🚀
