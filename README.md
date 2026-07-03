# 🔒 SPS — Sandbox Pentest Server

> *A Raspberry Pi-based portable pentesting sandbox — your own private hacking lab, no external platforms needed.*

![Status](https://img.shields.io/badge/status-in%20development-dc143c?style=flat-square)
![Hardware](https://img.shields.io/badge/hardware-Raspberry%20Pi-C51A4A?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-39ff14?style=flat-square)

---

## What is SPS?

SPS (Sandbox Pentest Server) is a self-hosted, portable penetration testing lab running on a **Raspberry Pi**. It provides a fully isolated environment with intentionally vulnerable machines and services — so you can practice real-world pentesting techniques **without relying on platforms like HackTheBox, TryHackMe, or VulnHub**.

Everything runs locally. No internet required. No accounts. No subscriptions.

---

## Why SPS?

| External Platforms | SPS |
|---|---|
| Requires internet connection | 100% offline capable |
| Monthly subscription | Free, one-time hardware cost |
| Limited to their machines | Build your own vulnerable environments |
| Can't customize targets | Full control over every service |
| Data goes through their servers | Completely private |

---

## Architecture

```
┌─────────────────────────────────────┐
│         Raspberry Pi (SPS Host)     │
│                                     │
│  ┌─────────┐  ┌─────────────────┐  │
│  │ Kali    │  │ Vulnerable VMs  │  │
│  │ Attacker│  │ (Docker)        │  │
│  │ Machine │  │                 │  │
│  └────┬────┘  │ • DVWA          │  │
│       │       │ • Metasploitable│  │
│       │       │ • Custom targets│  │
│       └──────▶│                 │  │
│               └─────────────────┘  │
│                                     │
│  Isolated network — no internet     │
└─────────────────────────────────────┘
         │
         ▼
   [Your laptop / tablet]
   Connect via WiFi hotspot
   Access web dashboard
```

---

## Features

- 🐳 Docker-based vulnerable environments (easy to spin up/down)
- 🌐 Isolated network — completely sandboxed
- 📡 Built-in WiFi hotspot to connect your attack machine
- 🎯 Pre-loaded targets: DVWA, Metasploitable, custom configs
- 📋 Built-in note-taking and progress tracking
- 🔋 Battery-powered, take it anywhere
- 🖥️ Web dashboard for managing lab environments

---

## Included Vulnerable Environments

| Target | Vulnerabilities | Difficulty |
|---|---|---|
| DVWA | SQLi, XSS, CSRF, File Upload, Command Injection | Beginner → Advanced |
| Metasploitable 2 | 20+ exploitable services | Intermediate |
| Custom Web App | OWASP Top 10 | Intermediate |
| Custom API | API security flaws | Intermediate |
| Custom Network | Misconfigured services | Advanced |

---

## Hardware Requirements

| Component | Spec |
|---|---|
| Board | Raspberry Pi 4 (8GB recommended) |
| Storage | 64GB+ microSD |
| Power | USB-C (power bank for portable use) |
| Network | Built-in WiFi + optional USB adapter |

---

## Quick Start

```bash
git clone https://github.com/Ely-Retr0/SPS
cd SPS
chmod +x install.sh
sudo ./install.sh
# Follow the setup wizard
sudo python sps.py --start
```

Then connect to the `SPS-Lab` WiFi network from your attack machine.

---

## ⚠️ Legal Disclaimer

SPS is intended for **educational purposes and authorized security practice only**. All vulnerable environments are intentionally designed for learning. Never deploy these on a public network.

---

## Author

**Elias Diaz Gutierrez** — [@Ely-Retr0](https://github.com/Ely-Retr0)  
*Think outside the fierrewall*
