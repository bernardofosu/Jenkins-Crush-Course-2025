# 🖥️ Jenkins Master + Agents Setup on AWS EC2

## 1️⃣ Create the Slave VM

On **AWS EC2**:

* Launch new instance(s) for agents (e.g., `Agent-1`, `Agent-2`).
* Use **Ubuntu** or **Amazon Linux 2**.
* After login via SSH:

  ```bash
  sudo apt update -y
  mkdir -p /home/ubuntu/jenkins
  cd /home/ubuntu/jenkins
  ```
* This directory (`/home/ubuntu/jenkins`) will be the **Remote root directory** for Jenkins agent.

---

## 2️⃣ Create a New Node in Jenkins

* Go to **Manage Jenkins → Nodes → New Node**.
* Enter name (e.g., `Agent-1`).
* Select **Permanent Agent** → Create.
* Configure:

  * 🔢 **Executors:** Match the number of vCPUs on the VM.
  * 📂 **Remote root directory:** `/home/ubuntu/jenkins`
  * 🏷️ **Labels:** `agent-1`
  * ⚡ **Usage:** Only build jobs with this label.
  * 🔑 **Launch method:** Launch agent via SSH.
  * 🌍 **Host:** Public IP of Agent-1.
  * 🔐 **Credentials:** select the SSH key you’ll add next.

---

## 3️⃣ Create Credentials 🔑

* Go to **Manage Jenkins → Credentials → Global**.
* Click **Add Credentials**.
* Select: **SSH Username with private key**.
* Username: `ubuntu`
* Paste the **.pem private key** used for EC2.
* Save as `ssh-key`.

⚡ **Difference: System vs Global**

* 🌍 **Global (Recommended):** Available to all Jenkins jobs, nodes, and agents → use for agent SSH keys.
* 🛠️ **System:** Only available internally to Jenkins, not visible to jobs.

👉 For EC2 agent connections → use **Global credentials**.

---

## 4️⃣ Finish Node Config

* Back in **Node settings**:

  * Host: `52.xx.xx.xx` (agent’s public IP).
  * Credentials: select `ssh-key`.
  * Host Key Verification Strategy: `Non verifying` for testing → in production prefer `Known hosts file`.
* Save ✅

---

## 5️⃣ Repeat for Agent-2

* Node name: `Agent-2`.
* Remote root directory: `/home/ubuntu/jenkins`.
* Labels: `agent-2`.
* Public IP: of Agent-2 VM.
* Use the same **Global credential** if both agents share the same `.pem` key.

✅ Now Jenkins Master connects to **Agent-1** and **Agent-2** via SSH.
✅ Jobs with label `agent-1` run on Agent-1 VM, and jobs with label `agent-2` run on Agent-2 VM.

---

## ⚙️ Executors Best Practice

* **Executor = number of concurrent jobs a node can run.**
* Rule: **Executors ≈ number of CPU cores** (sometimes minus 1 for overhead).

🔹 **Agents (Slaves):**

* Example: `t2.micro` (1 vCPU) → Executors = `1`.
* Example: `t2.medium` (2 vCPUs) → Executors = `2`.

🔹 **Master (Controller):**

* Always set **Executors = 0** → Master does not run jobs.
* Keeps the Master focused on **orchestration & monitoring**, not builds.

---

## ✅ Example Setup

* 🧠 **Master (Controller):** Executors = `0`
* 💻 **Agent-1 (t2.micro, 1 vCPU):** Executors = `1`
* 💻 **Agent-2 (t2.medium, 2 vCPUs):** Executors = `2`
