# 🌐 Jenkins Agent Ping Tests

Adding a **ping test step** is a neat way to make sure your Jenkins **master/controller** can actually reach the agents (or that agents can see each other if you’re doing distributed builds).

Here’s how you’d do it in **separate `Execute shell` blocks** in a Freestyle job:

---

## 1️⃣ Ping a Specific Agent’s Private IP
General
```bash
#!/bin/bash
set -euxo pipefail

# Replace with the agent’s private IP
AGENT_IP="172.31.28.101"

echo "Pinging agent at $AGENT_IP..."
ping -c 4 "$AGENT_IP"
```

Pin All
```bash
#!/bin/bash
set -euxo pipefail

# Replace with the agent’s private IP
Master_Private_IP="172.31.18.129"
AGENT1_Private_IP="172.31.20.163"
AGENT2_Private_IP="172.31.23.220"

# Master
echo "Pinging agent at $Master_Private_IP..."
ping -c 4 "$Master_Private_IP"

echo "Pinging agent at $AGENT1_Private_IP..."
ping -c 4 "$AGENT1_Private_IP"

echo "Pinging agent at $AGENT2_Private_IP..."
ping -c 4 "$AGENT2_Private_IP"
```

➡️ **`-c 4`** → send 4 packets and stop (so the build doesn’t hang forever).
⚠️ If the host is unreachable, this step will **fail the build** (good for health checks).

---

## 2️⃣ Ping Multiple Agents (loop)

```bash
#!/bin/bash
set -euxo pipefail

AGENTS=("172.31.28.101" "172.31.30.52" "172.31.33.7")

for ip in "${AGENTS[@]}"; do
  echo "Pinging $ip ..."
  if ping -c 3 "$ip"; then
    echo "$ip is reachable ✅"
  else
    echo "$ip is NOT reachable ❌"
    exit 1
  fi
done
```

---

## 3️⃣ Ping by Hostname (if agents have DNS names)

```bash
#!/bin/bash
set -euxo pipefail
ping -c 4 agent-1.local
```

---

## 🧩 How this helps in Jenkins

✨ Before trying to install or run anything, you confirm **connectivity**.
🔒 If you’re in **AWS**, pinging private IPs only works if your **security groups** allow ICMP traffic.
⚙️ Sometimes you’ll need to add a rule like:

* **Type**: ICMP 🛰️
* **Source**: VPC CIDR or master’s private IP 🏠
