# 🐳 Jenkins Master–Agent with Docker — Best‑Practice Notes (Curated)

A streamlined, **Docker‑first** setup for Jenkins Controller (Master) + Agents using **WebSocket** connections and **persistent storage**. This guide merges and refines the best bits from your notes, focusing on reliability, clean networking, and painless permissions.

---

## ✅ Prerequisites

* 🐧 Linux host with **Docker** installed
* 🌐 Basic terminal & networking knowledge
* ❗ No Java required on the host (images include what they need)

> 💡 Prefer **Docker named volumes** for persistence—they solve most ownership/permission headaches automatically.

---

## 1) Create an Isolated Network 🌐

```bash
docker network create --attachable jenkins-net
```

Verify:

```bash
docker network inspect jenkins-net | grep Name
```

> 🧭 Using an **attachable** network makes it easy to connect/disconnect containers for troubleshooting.

---

## 2) Create Persistent Storage 📦

### Recommended: Docker Volumes (production)

```bash
docker volume create jenkins_home
docker volume create jenkins_agent_vol
```

### Optional: Host Bind Mount (dev)

```bash
sudo mkdir -p /var/jenkins_agent
sudo chown -R 1000:1000 /var/jenkins_agent
sudo chmod -R 775 /var/jenkins_agent
```

> 🔐 Volumes handle UID/GID cleanly. Bind mounts may require `chown/chmod` (agent runs as UID 1000).

---

## 3) Run Jenkins Controller (Master) 🧠

```bash
docker run -d --name jenkins-master \
  --restart unless-stopped \
  --network jenkins-net \
  -v jenkins_home:/var/jenkins_home \
  -p 8080:8080 \
  jenkins/jenkins:lts
```

Unlock password:

```bash
docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
```

Open UI: `http://<host-ip>:8080`

> 🧵 **Classic inbound TCP (JNLP)** needs `-p 50000:50000` **and** firewall open on 50000. **WebSocket** does **not**; it tunnels over 8080.

---

## 4) Create an Agent in Jenkins UI 🧩

`Manage Jenkins → Nodes → New Node → Permanent Agent`

* Name: `node-1`
* Remote root / WorkDir: `/home/jenkins/agent`
* Launch method: **WebSocket** ("Launch agent by connecting it to the controller")
* Copy the **agent secret** 🔑

> 🧭 Also set **Manage Jenkins → System → Jenkins Location → Jenkins URL** to: `http://jenkins-master:8080/`

---

## 5) Run Jenkins Agent (WebSocket, with volume) 🚀

**Preferred (Docker volume):**

```bash
docker run -d --name jenkins-agent-1 \
  --restart unless-stopped \
  --network jenkins-net \
  -v jenkins_agent_vol:/home/jenkins/agent \
  jenkins/agent \
  java -jar /usr/share/jenkins/agent.jar \
  -url http://jenkins-master:8080/ \
  -secret <SECRET> \
  -name "node-1" \
  -webSocket \
  -workDir "/home/jenkins/agent"
```

**Alternative (host bind mount):**

```bash
docker run -d --name jenkins-agent-1 \
  --restart unless-stopped \
  --network jenkins-net \
  -v /var/jenkins_agent:/home/jenkins/agent \
  jenkins/agent \
  java -jar /usr/share/jenkins/agent.jar \
  -url http://jenkins-master:8080/ \
  -secret <SECRET> \
  -name "node-1" \
  -webSocket \
  -workDir "/home/jenkins/agent"
```

> ❌ Avoid hard‑coding container IPs (e.g., `172.21.0.2`). Use the **service name** `jenkins-master` on the shared network.

---

## 6) Verify & Operate ✅

Check containers:

```bash
docker ps
```

Agent logs:

```bash
docker logs -f jenkins-agent-1
```

DNS check from agent:

```bash
docker exec -it jenkins-agent-1 getent hosts jenkins-master
```

Jenkins UI: `Manage Jenkins → Nodes → node-1 → Online`

---

## 7) Troubleshooting 🛠️

**Agent keeps restarting (Exit 1):**

* 🔑 Secret wrong → Re-copy from node config.
* 🌐 URL wrong → Ensure `http://jenkins-master:8080/` and controller is resolvable on `jenkins-net`.
* 📁 Permission denied → If binding host dir, ensure:

  ```bash
  sudo chown -R 1000:1000 /var/jenkins_agent
  sudo chmod -R 775 /var/jenkins_agent
  ```
* 🧭 Network alias fix (force DNS names):

  ```bash
  docker network disconnect jenkins-net jenkins-agent-1 || true
  docker network connect --alias jenkins-master jenkins-net jenkins-master
  docker network connect --alias jenkins-agent-1 jenkins-net jenkins-agent-1
  ```
```sh
docker exec -it jenkins-agent-1 getent hosts jenkins-master
```
If it returns an IP (e.g., 172.21.0.2 jenkins-master), then the DNS resolution is working 🎉

If not, then either:
- The jenkins-master container is not on the same network, or
- You misspelled the container name, or
- The agent isn’t connected to jenkins-net.

### How internal DNS works on a user-defined bridge network (like your jenkins-net) 👇
- 🧭 Automatic DNS: Docker runs an internal DNS server for every user-defined network. Any container you attach with --network jenkins-net becomes reachable by its container name (and any --alias you set).

- 🔗 Your case: Since both jenkins-master and jenkins-agent-1 are on jenkins-net, the agent can use
http://jenkins-master:8080/ and Docker DNS will resolve jenkins-master to its current IP automatically (no hard-coding needed).

- 🔁 IPs can change; names won’t: If the controller restarts and gets a new IP, the name jenkins-master still resolves to the new IP—no config changes required.

🧪 Test resolution:
```sh
docker exec -it jenkins-agent-1 getent hosts jenkins-master
```

Gotchas / tips
- Use user-defined networks, not the default bridge (the default doesn’t do name resolution unless you use legacy --link).
- You can add extra names with an alias:
docker network connect --alias jenkins jenkins-net jenkins-master
- From the host, names don’t work—use published ports (e.g., http://localhost:8080) because Docker’s DNS is only inside the network.
- With Compose, you typically refer to the service name (e.g., http://jenkins-master:8080) and get the same DNS behavior.

**Switching to classic TCP (non‑WebSocket):**

```bash
# re-run controller with TCP port exposed
-p 8080:8080 -p 50000:50000
# open firewall
sudo ufw allow 50000/tcp && sudo ufw reload
```

Validate in Jenkins UI (Agent → Log) that it connects without WebSocket.

**Permission model explained:**

* The agent image runs as user `jenkins` (UID 1000). Docker matches **UID numbers**, not names.
* Named volumes avoid ownership conflicts; bind mounts may need `chown` to `1000:1000`.

---

## 8) Handy Commands 🧰

Restart containers:

```bash
docker restart jenkins-master jenkins-agent-1
```

Teardown:

```bash
docker rm -f jenkins-agent-1 jenkins-master
# preserve volumes   (remove if you really want)
# docker volume rm jenkins_home jenkins_agent_vol
```

Pull latest LTS:

```bash
docker pull jenkins/jenkins:lts
```

---

## 9) (Optional) Docker Compose 🧾

```yaml
version: "3.8"
services:
  jenkins-master:
    image: jenkins/jenkins:lts
    container_name: jenkins-master
    restart: unless-stopped
    networks: [jenkins-net]
    ports:
      - "8080:8080"
      # - "50000:50000"  # only if using classic TCP agents
    volumes:
      - jenkins_home:/var/jenkins_home

  jenkins-agent-1:
    image: jenkins/agent
    container_name: jenkins-agent-1
    restart: unless-stopped
    networks: [jenkins-net]
    volumes:
      - jenkins_agent_vol:/home/jenkins/agent
    command: >
      java -jar /usr/share/jenkins/agent.jar
      -url http://jenkins-master:8080/
      -secret ${SECRET}
      -name node-1
      -webSocket
      -workDir /home/jenkins/agent

networks:
  jenkins-net:
    driver: bridge
    attachable: true

volumes:
  jenkins_home:
  jenkins_agent_vol:
```

---

## 10) Final Notes 🧭

| Option                 | Use Case   | Pros                             | Cons                             |
| ---------------------- | ---------- | -------------------------------- | -------------------------------- |
| **Docker volume** 🔒   | Production | Auto permissions, portable, safe | Harder shell access to raw files |
| **Host bind mount** 🔥 | Dev        | Easy host inspection             | Requires manual `chown/chmod`    |

**Default to WebSocket** (fewer ports, simpler firewalls). Only enable port **50000** if you must use classic inbound TCP.

---

### ✅ Outcome

A stable Jenkins controller + agent setup with clean DNS, persistent workspaces, and minimal friction around permissions—ready for CI/CD.
