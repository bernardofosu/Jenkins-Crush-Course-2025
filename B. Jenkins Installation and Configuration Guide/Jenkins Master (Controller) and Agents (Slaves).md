# 🤖 Jenkins Master (Controller) and Agents (Slaves)

## 🧠 Jenkins Master (Controller)

* Central brain of Jenkins.
* Responsibilities:

  * Stores configurations (jobs, pipelines, plugins, credentials).
  * Provides the **Jenkins Web UI**.
  * Distributes jobs to agents.
  * Monitors agent status (online/offline).

📌 In the diagram: **Jenkins Master** delegates **Job-1** → Agent-1, **Job-2** → Agent-2.

---

## 🧩 Jenkins Agents (Slaves)

* Worker machines (VMs, containers, or physical servers).
* Responsibilities:

  * Execute jobs assigned by the master.
  * Return results (success/failure, logs, artifacts).
* Agents can have **labels** (e.g., `linux`, `docker`, `windows`) so jobs match the right environment.

📌 In the diagram:

* **VM-1 (4 GB)** → `agent-1` runs `Job-1`.
* **VM-2 (4 GB)** → `agent-2` runs `Job-2`.

---

## ⚙️ Workflow

1. Developer pushes code → GitHub/GitLab.
2. Jenkins Master detects event (webhook/poll).
3. Master assigns job → matching agent.
4. Agent executes the job (build/test/package).
5. Agent sends results back to Master.
6. Jenkins UI displays job results.

---

## 🚀 Why Use Agents?

* **Scalability** → Multiple agents handle jobs in parallel.
* **Isolation** → Jobs run in different environments.
* **Efficiency** → Master controls, agents execute.

---

✅ **Summary:**

* **Master = Brain 🧠** → Orchestrates, schedules, manages.
* **Agents = Muscle 💪** → Do the actual work.
* Distributed builds improve performance, reliability, and flexibility.
