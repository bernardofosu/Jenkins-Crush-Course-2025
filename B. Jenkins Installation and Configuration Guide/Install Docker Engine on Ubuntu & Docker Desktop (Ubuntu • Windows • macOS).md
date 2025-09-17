# 🐳✨ Install Docker Engine on Ubuntu & Docker Desktop (Ubuntu • Windows • macOS)

> Quick, emoji‑friendly guide with copy‑paste commands and official links.

---

## 🔗 Official Docs

* 🐧 **Docker Engine on Ubuntu:** [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
* 🪟 **Docker Desktop for Windows:** [https://docs.docker.com/desktop/setup/install/windows-install/](https://docs.docker.com/desktop/setup/install/windows-install/)
* 🍎 **Docker Desktop for macOS:** [https://docs.docker.com/desktop/setup/install/mac-install/](https://docs.docker.com/desktop/setup/install/mac-install/)

---

## ✅ Prerequisites (Ubuntu)

* 64‑bit Ubuntu: **Oracular 24.10**, **Noble 24.04 LTS**, **Jammy 22.04 LTS**
* Supported arch: `x86_64/amd64`, `armhf`, `arm64`, `s390x`, `ppc64le`
* ⚠️ **Firewall & iptables notes**

  * Exposed Docker ports can **bypass ufw/firewalld** rules → read: *Docker and ufw*
  * Use **iptables-nft** or **iptables-legacy** rules (not raw `nft`) and prefer rules in **`DOCKER-USER`** chain

> ℹ️ Ubuntu derivatives (e.g., Linux Mint) aren’t officially supported (may still work).

---

## 🧹 Uninstall Conflicting Packages (if any)

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove -y $pkg || true
done
```

> Your images/volumes in `/var/lib/docker/` are **not** removed by this.

---

## 🛠️ Install Docker Engine (APT repository) — Recommended

### 1) Set up Docker’s APT repo

```bash
# Add Docker’s official GPG key
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

### 2) Install the Docker packages (latest stable)

```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 3) Verify 🎉

```bash
sudo docker run hello-world
```

You should see a confirmation message.

> 💡 **Non‑root use**: add your user to the `docker` group (optional, convenience)

```bash
# allow current user to run docker without sudo
sudo usermod -aG docker $USER
newgrp docker
# test
docker run hello-world
```

---

## 📦 Install from .deb Packages (offline/manual)

1. Browse: `https://download.docker.com/linux/ubuntu/dists/` → your Ubuntu → `pool/stable/<arch>/`
2. Download these:

   * `containerd.io_<version>_<arch>.deb`
   * `docker-ce_<version>_<arch>.deb`
   * `docker-ce-cli_<version>_<arch>.deb`
   * `docker-buildx-plugin_<version>_<arch>.deb`
   * `docker-compose-plugin_<version>_<arch>.deb`
3. Install:

```bash
sudo dpkg -i ./containerd.io_<version>_<arch>.deb \
  ./docker-ce_<version>_<arch>.deb \
  ./docker-ce-cli_<version>_<arch>.deb \
  ./docker-buildx-plugin_<version>_<arch>.deb \
  ./docker-compose-plugin_<version>_<arch>.deb
```

4. Start & test:

```bash
sudo service docker start
sudo docker run hello-world
```

---

## ⚡ Convenience Script (dev/test only)

> **Not for production**. Always review scripts before running.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run   # preview
sudo sh ./get-docker.sh             # install latest stable
```

Pre‑release channel:

```bash
curl -fsSL https://test.docker.com -o test-docker.sh
sudo sh test-docker.sh
```

---

## 🧽 Uninstall Docker Engine

```bash
sudo apt-get purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
sudo rm -rf /var/lib/docker /var/lib/containerd
sudo rm -f /etc/apt/sources.list.d/docker.list /etc/apt/keyrings/docker.asc
```

---

# 🖥️ Docker Desktop

> Docker Desktop bundles Docker Engine, Compose v2, Kubernetes (optional), and UX tools.

## 🐧 Install Docker Desktop on Ubuntu

**Prereqs**

* Ubuntu **22.04 / 24.04 / latest non‑LTS** on **x86‑64**
* If not on GNOME, install terminal integration:

```bash
sudo apt install -y gnome-terminal
```

**Install**

1. Set up Docker’s APT repo (see **Install Docker Engine → Step 1** above).
2. Download the latest `docker-desktop-amd64.deb` (see *Release notes*).
3. Install:

```bash
sudo apt-get update
sudo apt-get install ./docker-desktop-amd64.deb
```

> You may see an `_apt` permission warning — it’s safe to ignore for this flow.

**Launch**

* GUI: open **Docker Desktop** from your app menu and **Accept** the terms
* CLI:

```bash
systemctl --user start docker-desktop
```

Set auto‑start:

```bash
systemctl --user enable docker-desktop
```

Stop:

```bash
systemctl --user stop docker-desktop
```

**Notes**

* Desktop creates its own **Docker context** to avoid clashing with any local engine
* Installs **Compose v2** and a wrapper CLI (`/usr/local/bin/com.docker.cli`)

**Check versions**

```bash
docker compose version
docker --version
docker version
```

**Upgrade**

* Download the new `.deb`, then:

```bash
sudo apt-get install ./docker-desktop-amd64.deb
```

---

## 🪟 Install Docker Desktop on Windows (link)

* 📥 Guide: [https://docs.docker.com/desktop/setup/install/windows-install/](https://docs.docker.com/desktop/setup/install/windows-install/)
* 🧩 Requires Windows 10/11 with WSL 2 (recommended) or Hyper‑V
* 🧰 Includes Docker Engine, Compose v2, Kubernetes (optional), Docker Desktop UI

## 🍎 Install Docker Desktop on macOS (link)

* 📥 Guide: [https://docs.docker.com/desktop/setup/install/mac-install/](https://docs.docker.com/desktop/setup/install/mac-install/)
* 💻 Intel (x86‑64) and Apple Silicon (M1/M2/M3) builds available
* 🧰 Includes Docker Engine, Compose v2, Kubernetes (optional), Docker Desktop UI

---

## 🧭 Next Steps

* 🔐 Review **post‑installation** steps for Linux (manage the `docker` group)
* 🧱 Try the **Docker workshop** (build/run your first image)
* 📚 Explore **Compose**, **Buildx**, and **Kubernetes** from Desktop

> Need a **Docker Compose** starter for Jenkins or a quick **GPU** setup? I can add templates here. 🚀
