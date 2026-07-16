#  SPS — Sandbox Pentest Server 

> *A Raspberry Pi-based portable pentesting sandbox — your own private hacking lab, no external platforms needed.*

![Status](https://img.shields.io/badge/status-active-39ff14?style=flat-square)
![Hardware](https://img.shields.io/badge/hardware-Raspberry%20Pi%204-C51A4A?style=flat-square)
![Docker](https://img.shields.io/badge/orchestration-Docker-2496ED?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-00599C?style=flat-square)
![OS](https://img.shields.io/badge/OS-Debian%2012%20ARM64-A81D33?style=flat-square)

---

## What is SPS?

**SPS (Sandbox Pentest Server)** is a self-hosted, portable penetration testing lab running on a **Raspberry Pi 4**. It provides a fully isolated environment with intentionally vulnerable machines and services — so you can practice real-world pentesting techniques **without relying on external platforms**.

By configuring the Raspberry Pi as an independent Wireless Access Point (**Bunker Mode**), it creates a closed network loop. Everything runs locally. No internet required. No accounts. No subscriptions.

---

##  Why SPS?

| Feature | External Platforms | SPS |
|---|---|---|
| Internet | Required | ❌ Not needed |
| Cost | Monthly subscription | ✅ One-time hardware |
| Targets | Limited to theirs | ✅ Build your own |
| Customization | Restricted | ✅ Full control |
| Privacy | Data through their servers | ✅ 100% local |
| Portability | Cloud-dependent | ✅ Fits in your bag |

---

##  Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────  ┐
│               Raspberry Pi 4 — SPS Host                                           │
│  [ OS: Debian 12 / Raspberry Pi OS Lite ARM64 ]  [ RAM: 8GB ]  [ SD: 128GB ]      │
│                                                                                   │
│  📡 WIRELESS INTERFACE (RaspAP — Bunker Mode)                                     │
│  ├─ SSID:    RaspAP (customizable)                                                │
│  ├─ Subnet:  10.3.141.0/24                                                        │
│  └─ Gateway: 10.3.141.1  (static)                                                 │
│                                                                                   │
│  🐳 DOCKER ENGINE                                                                 │
│  └─ Internal Bridge Network: lab-net                                              │
│      ├─ Portainer CE   → :9000  (management dashboard)                            │
│      ├─ OWASP JuiceShop→ :3000  (modern web app target)                           │
│      ├─ DVWA           → :8080  (classic vulnweb target)                          │
│      └─ MariaDB        → :3306  (internal only, no expose)                        │
└──────────────────────────▲────────────────────────────────────────────────────── ─┘
                           │
                           │  Direct Wi-Fi Connection
                           ▼
              💻 ATTACKER MACHINE
              [ Kali Linux / Parrot OS / BlackArch ]
```

---

##  Hardware Requirements

| Component | Specification |
|---|---|
| Board | Raspberry Pi 4 Model B (8GB RAM recommended) |
| Storage | 64GB+ microSD (128GB recommended) |
| OS | Debian 12 — Native ARM64 |
| Network | Built-in WiFi (no extra hardware needed) |
| Power | Official Pi 4 USB-C PSU (5V/3A) |

---

##  Installation Guide

> **Prerequisites:** Fresh Debian 12 install on the Pi, temporarily connected via **Ethernet** (with internet access) to download packages.

---

### Phase 1 — Base System

Update and install essential dependencies before anything else.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget git software-properties-common -y
```

---

### Phase 2 — Network Isolation (Bunker Mode)

Install **RaspAP** to turn the Pi into its own independent Wi-Fi router.

```bash
curl -sL https://install.raspap.com | bash
sudo reboot
```

> ⚠️ **After this reboot:** unplug the Ethernet cable. Connect to the Wi-Fi network **`raspi-webgui`** from your laptop and SSH in at `10.3.141.1`. From this point, all administration is wireless.

Post-reboot config (via RaspAP web panel at `http://10.3.141.1`):
- Set your custom SSID and password
- DHCP range: `10.3.141.10 – 10.3.141.100`
- Static IP confirmed: `10.3.141.1`

---

### Phase 3 — Docker Engine

Install Docker and create the isolated internal network for the lab containers.

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Allow your user to run Docker without sudo
sudo usermod -aG docker $USER
newgrp docker

# Create the internal bridge network for container communication
docker network create lab-net
```

---

### Phase 4 — Deploy the Lab

All containers use `--restart always` so the lab **boots automatically** every time you power on the Pi.

```bash
# 1. Portainer — Graphical container management dashboard
docker run -d \
  -p 8000:8000 -p 9000:9000 \
  --name=portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# 2. MariaDB — Backend database for DVWA
#    Security: assigned to lab-net only, zero external port exposure
docker run -d \
  --name lab-db \
  --network lab-net \
  --restart always \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=dvwa \
  -e MYSQL_USER=dvwa \
  -e MYSQL_PASSWORD=password \
  mariadb:latest

# 3. DVWA — Damn Vulnerable Web App (SQLi, XSS, brute force, etc.)
docker run -d \
  --name lab-dvwa \
  --network lab-net \
  -p 8080:80 \
  --restart always \
  ghcr.io/digininja/dvwa

# 4. OWASP Juice Shop — Modern vulnerable Node.js/Angular app
docker run -d \
  --name lab-juiceshop \
  -p 3000:3000 \
  --restart always \
  bkimminich/juice-shop
```

---

##  Accessing Your Lab

Connect to the Pi's Wi-Fi from your attacker machine, then open your browser:

| Service | URL | Default Credentials |
|---|---|---|
| RaspAP Panel | `http://10.3.141.1` | `admin` / `secret` |
| Portainer | `http://10.3.141.1:9000` | Set on first access |
| DVWA | `http://10.3.141.1:8080` | `admin` / `password` |
| Juice Shop | `http://10.3.141.1:3000` | Self-registration |
| SSH | `ssh admin@10.3.141.1` | Your system password |

---

##  Troubleshooting

<details>
<summary><b>I forgot my Portainer password and the Pi has no internet</b></summary>

Since all images are already stored locally on the Pi, you can nuke the Portainer container and volume and recreate it — no internet needed, no lab data lost.

```bash
# Destroy the current container
docker rm -f portainer

# Delete the volume with the old password
docker volume rm portainer_data

# Spin up a fresh instance
docker run -d \
  -p 8000:8000 -p 9000:9000 \
  --name=portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```
</details>

<details>
<summary><b>The RaspAP SSID doesn't appear after reboot</b></summary>

SSH into the Pi (connect via Ethernet temporarily) and check:

```bash
sudo systemctl status hostapd --no-pager
sudo systemctl status dnsmasq --no-pager
ip addr show wlan0
```

If hostapd is inactive:
```bash
sudo systemctl enable hostapd
sudo systemctl start hostapd
```
</details>

<details>
<summary><b>A container is not running after reboot</b></summary>

```bash
# Check all container statuses
docker ps -a

# Start a specific container manually
docker start lab-dvwa
docker start lab-juiceshop
docker start lab-db
docker start portainer
```
</details>

---

## Safe Shutdown

> The Pi runs its OS on a microSD card. **Never unplug it without shutting down first** — abrupt power loss can corrupt the filesystem.

```bash
sudo poweroff
```

Wait for the **green activity LED** on the Pi to stop blinking completely before disconnecting power.

---

## ⚠️ Legal Disclaimer

SPS is intended **strictly for educational purposes and authorized security research**. All vulnerable environments are intentionally designed for learning in a controlled, isolated setting.

**Never deploy these services on a public network. Never use these techniques on systems you do not own or have explicit written permission to test.**

The author assumes no responsibility for misuse of this project.

---

##  Author

**Elias Diaz Gutierrez** — [@Ely-Retr0](https://github.com/Ely-Retr0)

Think outside the firewall.
---

