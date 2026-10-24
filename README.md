<div align="center">

# 🛡️ LockWall

**Windows Server Brute-Force Protection**

*Lock down your Windows server like a real fortress.*

[![Version](https://img.shields.io/badge/version-2.9.6-blue.svg)](https://github.com/SMSerg2000/LockWall/releases)
[![Platform](https://img.shields.io/badge/platform-Windows%2010%2B%20%7C%20Server%202016%2B-lightgrey.svg)](https://www.microsoft.com/windows-server)
[![License](https://img.shields.io/badge/license-Freeware-brightgreen.svg)](LICENSE)
[![Installer downloads](https://img.shields.io/github/downloads/SMSerg2000/LockWall/LockWall-Setup-x64.msi)](https://github.com/SMSerg2000/LockWall/releases)

**[⬇️ Download](https://github.com/SMSerg2000/LockWall/releases/latest)** · **[🌐 lockwall.app](https://lockwall.app/)** · **[📖 Docs](docs/USER_GUIDE.md)**

</div>

---

## What is LockWall?

**LockWall** is a Windows service that protects your servers from brute-force attacks against remote access services. It monitors the Windows Event Log in real-time, detects failed login attempts, and automatically blocks attacking IPs via Windows Firewall.

### Protected services

- 🖥️ **RDP** (Remote Desktop Protocol) — Event ID 4625 in the Security Log
- 📧 **OWA** (Outlook Web Access) — IIS-hosted authentication via w3wp.exe
- 🗄️ **SQL Server** — Event ID 18456 in the Application Log
- 🔑 **SSH** (Windows OpenSSH Server) — Event ID 4 in the OpenSSH/Operational log

### Why LockWall?

RDP brute-force is one of the most common attacks on internet-facing Windows servers. A public-facing machine can receive thousands of automated login attempts a day. LockWall blocks attackers **before** they crack passwords — automatically, with progressive escalation, and with notifications.

**Free forever** — all four protocols, every feature, no license key, no seat limits.

---

## ✨ Features

### Detection
- 🔍 **RDP / OWA / SQL / SSH** brute-force monitoring — four protocols, one service
- 🎯 **Password Spray Detection** — blocks IPs trying multiple usernames
- 🐢 **Slow Brute Force Detection** — blocks addresses that keep trying below the regular limit (20 failed logons within 24 hours by default)
- 🌐 **Subnet Blocking** — when 3 addresses of one /24 are blocked within an hour, the whole /24 is blocked for a day, and longer each time it comes back when Progressive Blocking is on (public IPv4 only, never over the whitelist)
- 🔓 **Guessed-password alert** — a successful logon after 3 or more failed attempts within an hour is reported with the account, the address and the user names tried
- 📈 **Progressive Blocking** — escalating duration for repeat offenders (1h → 24h → 7d → permanent)
- ⚙️ **Per-source Policies** — separate thresholds for RDP, OWA, SQL
- ✅ **Whitelist** with CIDR support (block subnets, never block trusted networks)
- 🌍 **GeoIP** — optional country/city/ISP lookup for attacking IPs (multiple providers; off by default)
- 👁️ **Knows when it can't see** — warns if Windows stops writing the Security log or failed-logon auditing is switched off; one button fixes the log; it also tells you when successful logons are not audited, which the guessed-password alert needs
- ⏱️ **Boot catch-up** — failed logons from the minutes before the service starts are still counted
- 🛡️ **Microsoft Defender watch** — Defender detections and protection switches (real-time protection off, a weakened setting, Tamper Protection, an emptied Defender log) go to Telegram / Email; LockWall changes nothing in Defender
- 🔥 **Firewall state awareness** — a firewall switched off is noticed within ten seconds and reported, with a reminder every hour until it is back; a firewall that is on but lets inbound connections in by default is reported too, and one button switches it to Block safely (RDP allow rule first, automatic revert unless you confirm); a network profile the server is not using right now (say, Domain on a server outside a domain) is watched too

### Web Interface
- 📊 **Dashboard** — real-time statistics and top attackers
- 🚫 **Blocked IPs** — manage blocks (manual block/unblock, change duration, sort by address or block type)
- 📈 **Analytics** — graphs of hourly attempts, daily activity, top countries
- 📋 **Logs viewer** — live updates, filters, search
- ⚙️ **Settings** — edit config via the UI, no YAML editing required
- 🩺 **Health page** — system diagnostics (DB, firewall, notifications, disk, service)
- 📑 **Audit Log** — track all admin actions
- 👥 **User Management** — multi-admin with RBAC (Admin / Viewer roles)

### Integration & Automation
- 🔔 **Notifications** — Telegram (a private chat or a group) + Email on every block, with GeoIP enrichment; every Telegram message starts with an icon for its type and names the server with its public address; a weekly summary every Monday
- 📤 **CSV Export** — Blocked IPs, Audit Log, Login Attempts
- 🔑 **API Tokens** — Bearer / X-API-Key auth for the current JSON endpoints (and the future versioned public API)

### Architecture
- 🪟 **Native Windows Service** — lightweight, runs entirely on your server
- 🔥 **Smart Firewall Rules** — grouped by duration category (not one rule per IP)
- ☁️ **Self-hosted** — no cloud backend, no external agents; your data stays on your server
- 🔒 **Authentication** — login, RBAC, session timeout, forced password change
- 🗝️ **Encrypted secrets** — the Telegram token and the SMTP password in `config.yaml` are encrypted for the server (Windows DPAPI) and never shown in Settings

---

## 🚀 Quick Start

1. **Download** [`LockWall-Setup-x64.msi`](https://github.com/SMSerg2000/LockWall/releases/latest) (~13.7 MB)
2. **Run the installer** (accept the license) — it installs to `C:\LockWall` and registers + starts the Windows service automatically
3. **Open the web UI**: http://127.0.0.1:8880/
4. **Login**: `admin` / `lockwall` (you'll be forced to change the password)

That's it — LockWall is now watching your Event Log and blocking attackers.

### ⚠️ First-time setup — whitelist your network!

**Before exposing the server**, add your trusted networks to the whitelist
(Settings → Whitelist in the web UI, or `C:\LockWall\config\config.yaml`):

```yaml
whitelist:
  - 127.0.0.1
  - "::1"
  - 192.168.1.0/24      # Your local network
  - 10.0.0.0/8          # VPN
  - 203.0.113.5         # Your public IP
```

This prevents you from ever locking yourself out.

---

## 🎛️ Service Management

LockWall installs as a standard Windows service. Manage it from an elevated prompt:

```powershell
lockwall.exe status       # Check status
lockwall.exe stop         # Stop
lockwall.exe start        # Start
lockwall.exe restart      # Restart
lockwall.exe test-notify  # Send a test Telegram/Email notification

# View active firewall rules
netsh advfirewall firewall show rule name=all | findstr LOCKWALL_BLOCK
```

To uninstall, use **Add/Remove Programs** (or re-run the MSI).

---

## ⚙️ Configuration

Everything is editable in **Settings** in the web UI — no YAML required. Key options:

```yaml
detection:
  max_attempts: 5                # Failed attempts before block
  time_window_minutes: 10        # Counting window
  block_duration_minutes: 10080  # 7 days (0 = permanent)

progressive_blocking:
  enabled: true
  levels: [60, 1440, 10080, 0]   # 1h → 24h → 7d → permanent

spray_detection:
  enabled: true
  unique_usernames: 3            # 3+ usernames = spray attack

owa_protection:  { enabled: false }   # Enable on OWA servers
sql_protection:  { enabled: false }   # Enable on SQL Server hosts
ssh_protection:  { enabled: false }   # Enable on hosts with OpenSSH Server

web:
  host: 127.0.0.1                # Don't expose externally without HTTPS!
  port: 8880
```

Full reference: [`docs/USER_GUIDE.md`](docs/USER_GUIDE.md).

---

## 🔐 Trust & Verification

> ⚠️ **Download LockWall only from [lockwall.app](https://lockwall.app/) or this repository's [GitHub Releases](https://github.com/SMSerg2000/LockWall/releases).** Do not install repackaged copies from third-party mirrors or software catalogs.

LockWall is **closed-source freeware**. To let you trust a binary you can't read, every release is verifiable:

- ✅ **SHA-256 checksum** published with each release — verify your download matches
- ✅ **VirusTotal scan** linked in every release's notes
- ✅ **Open issue tracker** — bugs and questions in the public [Issues](https://github.com/SMSerg2000/LockWall/issues)

```powershell
# Verify your download (compare against the SHA-256 in the release notes)
Get-FileHash .\LockWall-Setup-x64.msi -Algorithm SHA256
```

Found a vulnerability? See [SECURITY.md](SECURITY.md) — please disclose responsibly.

---

## 📚 Documentation

| File | Audience | Language |
|------|----------|----------|
| [USER_GUIDE.md](docs/USER_GUIDE.md) | End users / administrators | English |
| [PHILOSOPHY.md](docs/PHILOSOPHY.md) | Why LockWall is free | English |

---

## 🛡️ Security Best Practices

1. ✅ **Change the default password** immediately (forced on first login)
2. ✅ **Whitelist your trusted networks** before exposing the service
3. ✅ **Keep `host: 127.0.0.1`** unless you put an HTTPS reverse proxy in front
4. ✅ **Back up** `config.yaml` and `data\lockwall.db` regularly

---

## 🌟 What's Coming Next

LockWall is **free and stays free** — no paid tier, no locked features, no license keys. The roadmap is simply about making the free product better:

- 🔏 **Code signing** — signed binaries so Windows stops warning "Unknown Publisher"
- 🔌 **Versioned REST API** — stable `/api/v1/…` endpoints for automation and SIEM
- 📡 **Webhooks** — push block events to external systems
- 🌍 **More notification channels** beyond Telegram / Email

Have an idea or hit a bug? **[Open an issue](https://github.com/SMSerg2000/LockWall/issues)** — feature requests and reports are welcome.

---

## 📋 Requirements

- **OS**: Windows 10 / Server 2016 (or newer), 64-bit
- **RAM**: 256 MB (512 recommended)
- **Disk**: 100 MB (500 MB for logs)
- **Privileges**: Administrator (for Windows Firewall + audit policy)
- **Audit Policy**: Logon Failure auditing (LockWall enables it automatically); Logon Success auditing for the guessed-password alert (one button on the Health page)

---

## 📄 License

**LockWall is proprietary freeware** — free to use for personal and internal commercial purposes, but **not** open source. Modification, reverse-engineering, resale, rebranding, and public redistribution (re-hosting or mirroring the installer) are not permitted — please download only from the official sources. See [LICENSE](LICENSE) (EULA) for the full terms.

© 2026 Serhii Smoktii. All rights reserved.

## 🌐 Links

- **Website**: [lockwall.app](https://lockwall.app/)
- **Download**: [GitHub Releases](https://github.com/SMSerg2000/LockWall/releases/latest)
- **Issues**: [github.com/SMSerg2000/LockWall/issues](https://github.com/SMSerg2000/LockWall/issues)

---

<div align="center">

**LockWall** — Lock down your Windows server like a real fortress. 🛡️

</div>
