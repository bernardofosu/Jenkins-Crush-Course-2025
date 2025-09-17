# 🔐 Complete Production Setup of HashiCorp Vault on EC2 with TLS & DNS

---

## ✅ Step 1: Install Required Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y unzip curl jq gnupg snapd

# Install Certbot
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

---

## ✅ Step 2: Install Vault (Official Repo)

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | \
  sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install vault -y
```

---

## ✅ Step 3: Create Vault User & Directories

```bash
sudo useradd --system --home /etc/vault.d --shell /bin/false vault
sudo mkdir -p /opt/vault/data
sudo mkdir -p /etc/vault.d
```

---

## ✅ Step 4: Generate TLS Cert with Let’s Encrypt

➡️ Ensure DNS **vault.skjptpp.in** points to the EC2 public IP.

```bash
sudo certbot certonly --standalone -d vault.skjptpp.in
```

📁 Certificates:

* `/etc/letsencrypt/live/vault.skjptpp.in/fullchain.pem`
* `/etc/letsencrypt/live/vault.skjptpp.in/privkey.pem`

### ❗️Fix: Vault Permission Denied on Let’s Encrypt Certs

```bash
sudo cp /etc/letsencrypt/live/vault.skjptpp.in/fullchain.pem /etc/vault.d/vault.crt
sudo cp /etc/letsencrypt/live/vault.skjptpp.in/privkey.pem  /etc/vault.d/vault.key

sudo chown vault:vault /etc/vault.d/vault.*
sudo chmod 640 /etc/vault.d/vault.key
sudo chmod 644 /etc/vault.d/vault.crt
```

---

## ✅ Step 5: Configure Vault – `/etc/vault.d/vault.hcl`

```hcl
storage "file" {
  path = "/opt/vault/data"
}

listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/etc/vault.d/vault.crt"
  tls_key_file  = "/etc/vault.d/vault.key"
}

ui = true
disable_mlock = true

api_addr     = "https://vault.skjptpp.in:8200"
cluster_addr = "https://172.31.42.120:8201"  # ← replace with EC2 private IP
```

---

## ✅ Step 6: systemd Service

```bash
sudo tee /etc/systemd/system/vault.service > /dev/null <<'EOF'
[Unit]
Description=Vault Server
Requires=network-online.target
After=network-online.target

[Service]
User=vault
Group=vault
ProtectSystem=full
ProtectHome=read-only
PrivateTmp=yes
PrivateDevices=yes
SecureBits=keep-caps
AmbientCapabilities=CAP_IPC_LOCK
Capabilities=CAP_IPC_LOCK+ep
CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
ExecStart=/usr/bin/vault server -config=/etc/vault.d/vault.hcl
ExecReload=/bin/kill --signal HUP $MAINPID
KillMode=process
Restart=on-failure
LimitNOFILE=65536
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
EOF
```

---

## ✅ Step 7: Start Vault

```bash
sudo systemctl daemon-reexec
sudo systemctl enable vault
sudo systemctl start vault
sudo systemctl status vault
```

---

## ✅ Step 8: Export `VAULT_ADDR`

```bash
export VAULT_ADDR=https://vault.skjptpp.in:8200
# Persist it
echo 'export VAULT_ADDR=https://vault.skjptpp.in:8200' >> ~/.bashrc
source ~/.bashrc
```

---

## ✅ Step 9: Initialize Vault

```bash
vault operator init -key-shares=5 -key-threshold=3
```

⚠️ **Store the 5 unseal keys & root token securely!**

---

## ✅ Step 10: Unseal Vault

```bash
vault operator unseal <key-1>
vault operator unseal <key-2>
vault operator unseal <key-3>

vault status
```

---

## ✅ Step 11: Login

```bash
vault login <initial-root-token>
```

---

## ✅ Step 12: Enable KV v2 & Try Secrets

```bash
vault secrets enable -path=secret kv-v2
vault kv put secret/myapp username=admin password=supersecure123
vault kv get secret/myapp
```

---

# 🔑 Create Read-Only Token & Policy for Jenkins

### 1) Login as root

```bash
vault login <root-token>
```

### 2) Policy: `jenkins-read-policy.hcl`

```hcl
path "secret/data/jenkins/*" {
  capabilities = ["read"]
}
```

Apply policy:

```bash
vault policy write jenkins-read jenkins-read-policy.hcl
```

### 3) Create short-lived token

```bash
vault token create -policy=jenkins-read -ttl=24h
```

### 4) Example secret

```bash
vault kv put secret/jenkins/docker username=mydockeruser password=MyP@ssword123
```

---

## 🤝 Jenkins Integration (Vault Plugin)

### Jenkinsfile (Simple Usage)

```groovy
pipeline {
  agent any
  environment { VAULT_ADDR = 'https://vault.skjptpp.in:8200' }
  stages {
    stage('Use Vault Secrets') {
      steps {
        script {
          withVault([
            vaultSecrets: [[
              path: 'secret/jenkins/docker',
              engineVersion: 2,
              secretValues: [
                [vaultKey: 'username', envVar: 'DOCKER_USERNAME'],
                [vaultKey: 'password', envVar: 'DOCKER_PASSWORD']
              ]
            ]],
            vaultCredentialId: 'vault-cred'
          ]) {
            sh '''
              echo "Username from Vault: $DOCKER_USERNAME"
              echo "Password from Vault: $DOCKER_PASSWORD"
            '''
          }
        }
      }
    }
  }
}
```

### Jenkinsfile (Global Usage of Secrets)

```groovy
pipeline {
  agent any
  environment {
    DOCKER_USERNAME = ''
    DOCKER_PASSWORD = ''
  }
  stages {
    stage('Fetch Secrets from Vault') {
      steps {
        script {
          withVault(
            configuration: [
              disableChildPoliciesOverride: false,
              engineVersion: 2,
              timeout: 60,
              vaultCredentialId: 'vault-token',
              vaultUrl: 'https://vault.skjptpp.in:8200'
            ],
            vaultSecrets: [[
              engineVersion: 2,
              path: 'secret/jenkins/docker',
              secretValues: [
                [envVar: 'DOCKER_USERNAME', vaultKey: 'username'],
                [envVar: 'DOCKER_PASSWORD', vaultKey: 'password']
              ]
            ]]
          ) {
            env.DOCKER_USERNAME = env.DOCKER_USERNAME
            env.DOCKER_PASSWORD = env.DOCKER_PASSWORD
            echo "Fetched DOCKER_USERNAME: ${env.DOCKER_USERNAME}"
          }
        }
      }
    }
    stage('Use Secrets') {
      steps { echo "Using secret outside: ${env.DOCKER_USERNAME}" }
    }
  }
}
```

> 🔒 **Tip:** Avoid echoing real secrets in production logs.

---

## 🔁 Bonus: Auto-Renew TLS Certs (Cron)

```bash
sudo crontab -e
# Daily at 03:00
0 3 * * * certbot renew --post-hook "systemctl restart vault"
```

---

## 🧪 Health Checks & Troubleshooting

* 🔎 `curl -sSk https://vault.skjptpp.in:8200/v1/sys/health` → verify status
* 🔐 403 during login? Check **policy** and **token TTL**.
* 🗝️ Unseal required on reboot if not using **Auto-Unseal** (e.g., with AWS KMS).
* 📜 Check logs: `journalctl -u vault -f`

---

## 🧩 Optional: Production Hardening

* 🧰 Use **Integrated Storage (raft)** for HA instead of file storage.
* 🔑 Enable **Auto-Unseal** with AWS KMS or Cloud KMS.
* 🛡️ Restrict Security Groups → allow 8200/8201 from trusted sources only.
* 🧱 Place behind an **ALB** with TLS and health checks.
* 📈 Enable Audit: `vault audit enable file file_path=/var/log/vault_audit.log`

---

🎉 You now have a **production-grade Vault** on EC2 with **TLS, DNS & Jenkins integration**! 🚀
