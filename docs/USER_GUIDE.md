# LockWall — User Guide

> **Version:** 2.1.0
> **Date:** 2026-07-12

---

## Table of Contents

1. [Introduction](#introduction)
2. [System Requirements](#system-requirements)
3. [Installation](#installation)
4. [First Login & Initial Setup](#first-login--initial-setup)
5. [Web Interface](#web-interface)
6. [Service Management](#service-management)
7. [Operations (Backup, Update, Uninstall)](#operations)
8. [Configuration](#configuration)
9. [Notifications](#notifications)
10. [GeoIP](#geoip)
11. [Detection Features](#detection-features)
12. [Firewall Rules](#firewall-rules)
13. [CSV Export](#csv-export)
14. [API Tokens](#api-tokens)
15. [Whitelist](#whitelist)
16. [Logs and Diagnostics](#logs-and-diagnostics)
17. [Troubleshooting](#troubleshooting)
18. [Security Recommendations](#security-recommendations)
19. [Before Production Checklist](#before-production-checklist)
20. [Privacy and Data Processing](#privacy-and-data-processing)
21. [FAQ](#faq)

---

## Introduction

**LockWall** is a Windows service that protects Windows Server from brute-force attacks against remote access services. It monitors Windows Event Log in real-time, detects failed login attempts, and automatically blocks attacking IP addresses via Windows Firewall.

### What LockWall Protects

- 🖥️ **RDP** (Remote Desktop Protocol) — Event ID 4625 in Security Log
- 📧 **OWA** (Outlook Web Access) — Event ID 4625 from IIS worker (w3wp.exe)
- 🗄️ **SQL Server** — Event ID 18456 in Application Log
- 🔑 **SSH** (Windows OpenSSH Server) — Event ID 4 in the OpenSSH/Operational log

### Why You Need It

RDP brute-force is the **#1 attack** on Windows servers. Bots scan the internet for open port 3389 and continuously attempt password guessing 24/7. Without protection, any Windows server with a public IP will receive **hundreds of thousands of login attempts per day**.

LockWall:
- Blocks attackers **before** they crack the password
- Reduces server load (attacks don't reach Windows authentication)
- Protects against **password spray** attacks (multiple usernames from one IP)
- Escalates block duration for repeat offenders (Progressive Blocking)
- Notifies administrators via Telegram and Email

### How It Works

```
[Attacker] → [Windows Event Log] → [LockWall Detector]
                                          ↓
                                 Threshold exceeded?
                                          ↓
                                 [Windows Firewall: BLOCK]
                                          ↓
                                 [Notification: Telegram/Email]
```

---

## System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| **OS** | Windows 10 / Server 2016 | Windows 11 / Server 2022 |
| **CPU** | 1 core | 2 cores |
| **RAM** | 256 MB | 512 MB |
| **Disk** | 100 MB | 500 MB (for logs) |
| **Privileges** | Administrator | Administrator |
| **Network** | — | Internet (for online GeoIP) |
| **Audit** | Logon auditing must be enabled |

LockWall automatically checks and enables (if needed):
- **Audit Logon** (Logon Failure) — for Event 4625
- **Audit Account Logon** — extended auditing

---

## Installation

### Quick Start

1. **Download** `LockWall-Setup-x64.msi` from the official GitHub Releases page:
   https://github.com/SMSerg2000/LockWall/releases/latest
   *(or from https://lockwall.app — do not use third-party mirrors)*
2. **Run the installer** as Administrator and accept the license. It will:
   - install LockWall to `C:\LockWall`;
   - register the Windows service named `LockWall`;
   - generate the default `config\config.yaml`;
   - start the service automatically.
3. **Open the web interface**: http://127.0.0.1:8880/
4. **Login**: `admin` / `lockwall`
5. **Change the password immediately** — LockWall forces this on first login.

### Step 1: Whitelist (CRITICAL!)

⚠️ **Add your network to the whitelist as soon as possible**, or you may lock yourself out!

Use **Settings → Whitelist** in the web UI, or edit `C:\LockWall\config\config.yaml`:

```yaml
whitelist:
  - 127.0.0.1
  - "::1"
  - 192.168.1.0/24      # Your local network
  - 10.0.0.0/8          # VPN
  - 203.0.113.5         # Your public IP
```

After changes — restart the service:
```powershell
lockwall.exe restart
```

---

## First Login & Initial Setup

### 1. Open Web Interface

After installation, open browser: **http://127.0.0.1:8880/**

### 2. Login

| Field | Default Value |
|-------|---------------|
| Username | `admin` |
| Password | `lockwall` |

### 3. Change Password (Forced)

On first login, LockWall **forces** redirect to change-password page. You cannot use the application without changing the default password — this protects against default credentials.

⚠️ Don't enter the same password `lockwall` as new — it will be rejected.

### 4. Configure Whitelist

If you haven't added your network yet — do it now via `config.yaml` or **Blocked IPs** → **Add to Whitelist**.

> **Note:** the session **SECRET_KEY is generated and persisted automatically** — no action needed. (Older guides mentioned a "Generate" button; it was removed once the key became automatic.)

---

## Web Interface

### Dashboard (`/`)

Main monitoring panel:
- **Statistics**: total attempts, blocked IPs, unique attackers
- **Top Attackers**: IPs with most attempts
- **Protection Status**: RDP / OWA / SQL modules active state
- **Recent Activity**: 24-hour graph

### Blocked IPs (`/blocks`)

IP block management:

| Column | Description |
|--------|-------------|
| IP / CIDR | Address or subnet |
| Country | Country (via GeoIP) |
| Source | RDP / OWA / SQL badge |
| Strikes | Number of times this IP was blocked |
| Attempts | Failed attempts count |
| Blocked At | Block timestamp |
| Unblock At | Auto-unblock time (or Permanent) |
| Actions | Unblock / Make Permanent / Make Temporary |

**Manual Block**: "Add Block" form — enter IP or CIDR (e.g., `109.205.213.0/24`), select duration (30 min, 1 hour, 6 hours, 24 hours, 7 days, permanent).

**CIDR Support**: block entire subnets with one rule.

### Analytics (`/stats`)

Attack statistics graphs (Chart.js):
- **Line chart**: hourly login attempts (last 24 hours)
- **Bar chart**: daily activity + unique IPs
- **Doughnut chart**: top-10 attacking countries

Period selectable from dropdown. Auto-refresh every 60 seconds.

### Logs (`/logs`)

Application log viewer (`logs/lockwall.log`):
- **Live updates** every 5-60 seconds (configurable)
- **Level filter**: ERROR (red), WARNING (yellow), INFO (blue)
- **Text search** with highlight
- **Efficient reading**: only last N lines

### Settings (`/settings`)

`config.yaml` editor (admin only).

Sections:
- **Detection** — thresholds, time window, block duration
- **Whitelist** — trusted IPs/CIDRs
- **Web Interface** — host and port
- **OWA Protection** — toggle (requires restart)
- **SQL Server Protection** — toggle (requires restart)
- **SSH Protection** — toggle (requires restart; needs Windows OpenSSH Server)
- **GeoIP** — online/local mode
- **Notifications** — Telegram, Email
- **Logging** — log level

After save, **Save & Restart Service** restarts the service.

### Health (`/health`)

System diagnostics:
- ✅ Database accessible
- ✅ Firewall sync (DB ↔ netsh)
- ✅ Notifications (Telegram + Email)
- ✅ Disk space
- ✅ Service status
- ✅ Uptime
- ✅ Version

### Audit (`/audit`)

Admin action log (admin only):
- Who did what (login, block, unblock, settings change)
- Filters: user, action, period
- CSV export

### Users (`/users`)

User management (admin only):
- Add/delete users
- Change role (Admin / Viewer)
- Reset password (with must_change_password flag)
- **API Tokens** — generate and revoke

### About (`/about`)

Version, description, changelog.

### Change Password (`/change-password`)

Always available from navbar (key icon).

---

## Service Management

The **MSI installer** registers, starts, and (when you uninstall via Add/Remove
Programs) removes the Windows service for you — most users never need these commands.
They are for advanced or manual control. All commands run as Administrator:

| Command | Action |
|---------|--------|
| `lockwall.exe status` | Status (Running/Stopped) |
| `lockwall.exe start` | Start service |
| `lockwall.exe stop` | Stop service |
| `lockwall.exe restart` | Restart service |
| `lockwall.exe test-notify` | Send a test Telegram/Email notification |
| `lockwall.exe run --web` | Console mode (debugging) |
| `lockwall.exe run --dry-run --web` | Test without real blocking |

**Advanced (manual install without the MSI):** `lockwall.exe install` registers the
service (Automatic — Delayed Start) and `lockwall.exe uninstall` removes it.

### View in services.msc

After installation, the service appears as:
- **Name**: `LockWall`
- **Display Name**: `LockWall - Windows Server Protection`
- **Startup Type**: Automatic (Delayed Start)

---

## Operations

### Backup

Recommended files to back up regularly:

- `C:\LockWall\config\config.yaml` — your settings (whitelist, thresholds, notifications)
- `C:\LockWall\data\lockwall.db` — blocked IPs, history, users, audit log
- *(optional)* `C:\LockWall\logs\` — application logs

To **restore**, stop the service, copy the files back, then start it:
```powershell
lockwall.exe stop
REM ...copy config.yaml / lockwall.db back into place...
lockwall.exe start
```

### Updating

Download the new release from the **official** [GitHub Releases](https://github.com/SMSerg2000/LockWall/releases) (verify its SHA-256), then run the new **`LockWall-Setup-x64.msi`**. The installer stops the old service, replaces the program, and starts the new version automatically — **your `config.yaml` and `data\lockwall.db` are preserved**. No need to uninstall first.

### Uninstall

Remove LockWall via **Settings → Apps** (or *Programs and Features*) → **LockWall** → **Uninstall**. This stops and removes the Windows service and deletes the program files.

The firewall rules are left in place by default. To remove them too:
```powershell
netsh advfirewall firewall delete rule name=LOCKWALL_BLOCK_30M
netsh advfirewall firewall delete rule name=LOCKWALL_BLOCK_1H
netsh advfirewall firewall delete rule name=LOCKWALL_BLOCK_6H
netsh advfirewall firewall delete rule name=LOCKWALL_BLOCK_24H
netsh advfirewall firewall delete rule name=LOCKWALL_BLOCK_7D
netsh advfirewall firewall delete rule name=LOCKWALL_BLOCK_PERMANENT
```
Your config and database remain in `C:\LockWall` — delete the folder if you no longer need them.

---

## Configuration

File: `C:\LockWall\config\config.yaml`

### Key Sections

#### `detection` — Detection parameters

```yaml
detection:
  max_attempts: 5
  time_window_minutes: 10
  block_duration_minutes: 10080  # 7 days (0 = permanent)
  logon_types: [3, 8, 10]        # 3=Network, 8=Cleartext (OWA), 10=RDP
  counter_mode: shared           # shared or separate
```

#### `progressive_blocking`

```yaml
progressive_blocking:
  enabled: true
  levels: [60, 1440, 10080, 0]  # 1h, 24h, 7d, permanent
```

#### `spray_detection`

```yaml
spray_detection:
  enabled: true
  unique_usernames: 3
  block_duration_minutes: 0    # 0 = permanent
```

#### `whitelist`

```yaml
whitelist:
  - 127.0.0.1
  - "::1"
  - 192.168.1.0/24
```

#### `web`

```yaml
web:
  enabled: true
  host: 127.0.0.1              # 0.0.0.0 for external (NOT RECOMMENDED)
  port: 8880
  # secret_key is generated and persisted automatically on first run — no need to set it
```

#### `auth`

```yaml
auth:
  enabled: true
  session_timeout_minutes: 30
```

#### `owa_protection` / `sql_protection` / `ssh_protection`

```yaml
owa_protection:
  enabled: false   # Enable only on servers with OWA

sql_protection:
  enabled: false   # Enable only on servers with SQL Server

ssh_protection:
  enabled: false   # Enable only on servers with Windows OpenSSH Server
```

> **SSH note:** attempts are read from the **OpenSSH/Operational** event log
> (`Failed password for … from IP port …`). The Security log (Event 4625) is
> not used for SSH — sshd does not report the client IP there.

⚠️ Changes to these require **service restart**.

---

## Notifications

### Telegram

#### Step 1: Create Bot
1. Find **@BotFather** in Telegram
2. Use `/newbot`
3. Enter name (e.g., `LockWall Server1`)
4. Get **Bot Token** (e.g., `123456:ABC-DEF...`)

#### Step 2: Get chat_id
1. Create group/channel, add bot
2. Send any message
3. Open `https://api.telegram.org/bot<TOKEN>/getUpdates`
4. Find `"chat":{"id":-100...}`

#### Step 3: Configure in LockWall

**Settings** → **Notifications** → **Telegram**:
- Bot Token, Chat ID, enable toggle
- **Send Test** to verify

### Email

**Settings** → **Notifications** → **Email**:
- SMTP Server (e.g., `smtp.gmail.com`)
- Port: `587`, Use TLS: ✅
- Username/Password
- From / To addresses

⚠️ For Gmail, use **App Passwords**, not main account password.

### Server Label

For multi-server setups:
```yaml
general:
  server_label: "PROD-EU-1"
```

---

## GeoIP

LockWall can geolocate attacker IPs. Pick the provider in **Settings → GeoIP**
(config key `geoip.mode`).

> **Default is `off`** — the safest, legally cleanest starting point. Enable a
> provider only when you actually want country/city/ASN enrichment.

> **GeoIP data is approximate.** Country, city and ASN can be wrong on mobile
> networks, VPNs, proxies and shared hosts. Treat GeoIP as context, not as a
> security decision on its own.

### Off Mode (`off`, default)

No geolocation lookups, nothing sent out. Safest starting point.

### ipinfo.io Lite Mode (`ipinfo`)

Uses **ipinfo.io** over HTTPS with your free API token:
- ✅ **Free Lite tier allows commercial use** (with attribution) — no local database needed
- 🔑 Requires a free token — register at [ipinfo.io/signup](https://ipinfo.io/signup)
- 📎 **Attribution required**: LockWall automatically shows "IP data powered by IPinfo" in the footer
- ℹ️ Lite returns country + ASN (city-level needs a paid plan)
- ℹ️ Attacker IP is sent to ipinfo.io
- ℹ️ Without a token, GeoIP is **disabled** — it does **not** fall back to ip-api.com

### Local Mode (`local`, MaxMind GeoLite2)

- ✅ No rate limits, works offline, **free for commercial use**, sends no IPs to any third party
- 🔑 Requires **your own .mmdb file** — LockWall does NOT ship the MaxMind database
  (GeoLite redistribution requires a separate MaxMind license)
- ℹ️ If the `.mmdb` file is missing, GeoIP is **disabled** — it does **not** fall back to ip-api.com

#### Setup

1. Register: https://www.maxmind.com/en/geolite2/signup
2. Download `GeoLite2-City.mmdb`
3. Copy to: `C:\LockWall\data\GeoLite2-City.mmdb`
4. **Settings** → **GeoIP** → Mode: **Local** → **Test**

### ip-api.com Free Mode (`online`)

Uses **ip-api.com**:
- ✅ No setup required
- ❌ Limit: 45 requests/minute · HTTP only (no HTTPS on the free endpoint)
- ❌ Attacker IP is sent to ip-api.com
- ⚠️ **Personal / non-commercial environments only** — the free endpoint forbids
  commercial use ([ip-api.com/docs/legal](https://ip-api.com/docs/legal)). On a
  company server use **ipinfo.io Lite**, **Local MaxMind**, or **Off**.

---

## Detection Features

### 1. Brute-force

```yaml
detection:
  max_attempts: 5
  time_window_minutes: 10
```

5 attempts in 10 minutes → block.

### 2. Password Spray

Detects when one IP tries many different usernames (instead of many passwords for one user). Immediate block.

```yaml
spray_detection:
  enabled: true
  unique_usernames: 3
```

3 unique usernames from one IP → spray → permanent block.

### 3. Progressive Blocking

```yaml
progressive_blocking:
  enabled: true
  levels: [60, 1440, 10080, 0]
```

| Strike | Duration |
|--------|----------|
| 1st | 1 hour |
| 2nd | 24 hours |
| 3rd | 7 days |
| 4th+ | Permanent |

Shown as **Strikes** badges in UI.

### 4. Per-source Policies

```yaml
detection:
  counter_mode: separate
  rdp:
    max_attempts: 5
    block_duration_minutes: 10080
  owa:
    max_attempts: 10            # OWA more lenient
  sql:
    max_attempts: 3             # SQL stricter
    block_duration_minutes: 0
```

---

## Firewall Rules

LockWall uses **6 grouped Windows Firewall rules** by duration category:

| Rule | Duration |
|------|----------|
| `LOCKWALL_BLOCK_30M` | 30 minutes |
| `LOCKWALL_BLOCK_1H` | 1 hour |
| `LOCKWALL_BLOCK_6H` | 6 hours |
| `LOCKWALL_BLOCK_24H` | 24 hours |
| `LOCKWALL_BLOCK_7D` | 7 days |
| `LOCKWALL_BLOCK_PERMANENT` | Forever |

All blocked IPs are stored in `remoteip=...` field (comma-separated). This allows blocking thousands of IPs without thousands of rules.

### View rules

```powershell
netsh advfirewall firewall show rule name=all | findstr LOCKWALL_BLOCK
netsh advfirewall firewall show rule name=LOCKWALL_BLOCK_7D
```

---

## CSV Export

| Source | Where | Contents |
|--------|-------|----------|
| Blocked IPs | `Blocked IPs` → **Export CSV** | IP, country, attempts, time |
| Audit Log | `Audit` → **Export CSV** | User actions |
| Login Attempts | API: `/api/export/attempts` | All login attempts |

CSV uses **UTF-8 with BOM** for Excel compatibility.

---

## API Tokens

LockWall provides REST API for integrations (SIEM, scripts, monitoring).

### Generate Token

1. Login as **admin**
2. **Users** → **API Tokens** → **Generate Token**
3. Enter name (e.g., `splunk-integration`)
4. Select role
5. ⚠️ **Copy token** — shown **only once!**

### Use Token

```bash
# Bearer header
curl -H "Authorization: Bearer YOUR_TOKEN" \
     http://127.0.0.1:8880/api/health

# X-API-Key header
curl -H "X-API-Key: YOUR_TOKEN" \
     http://127.0.0.1:8880/api/health
```

### Revoke

**Users** → **API Tokens** → **Revoke**.

### Security

- Stored as **SHA256 hash** (raw token never saved)
- Logged in Audit Log on creation/revocation
- `last_used` updated on each use
- Use **HTTPS** in production!

### API Status

LockWall exposes **JSON endpoints** used by the web UI and basic automation
(stats, health, logs, CSV export — see above). These are available in the current
release and authenticate via session **or** API token.

A **fully documented, versioned public REST API** (`/api/v1/…`) is planned for a future
release. Until those endpoints are officially documented and marked stable, **endpoint
compatibility between releases is not guaranteed** — pin to a LockWall version if you
automate against the current endpoints.

---

## Whitelist

Whitelist = IPs/subnets that will **never** be blocked.

```yaml
whitelist:
  - 127.0.0.1
  - "::1"
  - 192.168.0.0/16
  - 10.0.0.0/8
  - 203.0.113.5
```

✅ **Always add**:
- `127.0.0.1` and `::1`
- Local network
- Corporate VPN
- Office IP
- Public IP you administer from

⚠️ Without whitelist, you can lock yourself out!

---

## Logs and Diagnostics

- **Application log**: `C:\LockWall\logs\lockwall.log`
- **Rotation**: 10 MB max, 5 backups
- **UI**: `Logs` page

```yaml
logging:
  level: INFO  # DEBUG | INFO | WARNING | ERROR
```

---

## Troubleshooting

### Service won't start
1. Check logs: `C:\LockWall\logs\lockwall.log`
2. Check Event Viewer → Application → Source: `LockWall`
3. Run as **Administrator**
4. Check audit policy: `auditpol /get /subcategory:"Logon"`

### Web UI not accessible
1. Service running? `lockwall.exe status`
2. Port 8880 free? `netstat -an | findstr 8880`
3. Try different browser

### No blocks happening
1. Audit logon **must be enabled**
2. Check whitelist
3. Check `max_attempts` and `time_window_minutes`
4. Check logs for errors

### No notifications
- **Telegram**: Bot Token correct? Chat ID has minus sign for groups?
- **Email**: SMTP correct? Gmail uses App Password!
- Click **Send Test**

### "OWA detection not working"
- `owa_protection.enabled: true` in config.yaml
- Restart service
- Add `8` to `logon_types` (for OWA Forms-Based Auth)

### Emergency admin password recovery

Use this **only** if you have lost access to **all** admin accounts.

> ⚠️ Editing the SQLite database directly can corrupt LockWall data if done wrong.
> **Always back up the database first.**

1. Stop the service:
   ```powershell
   lockwall.exe stop
   ```
2. **Back up the database:**
   ```powershell
   copy C:\LockWall\data\lockwall.db C:\LockWall\data\lockwall.db.bak
   ```
3. Open `C:\LockWall\data\lockwall.db` with [DB Browser for SQLite](https://sqlitebrowser.org/).
4. Delete the locked/lost admin user from the `users` table.
5. Start the service:
   ```powershell
   lockwall.exe start
   ```
6. Log in with the default `admin` / `lockwall` and **immediately change the password**.

---

## Security Recommendations

### 🔒 Mandatory

1. **Change default password** — don't keep `admin/lockwall`
2. **Whitelist your network** — avoid lockout
3. **Don't expose port 8880** — keep `host: 127.0.0.1`

### 💡 Recommended

1. **HTTPS reverse proxy** (nginx, IIS, Caddy) for external access
2. **Backup `config.yaml`** and `data\lockwall.db`
3. **Multiple admins** (in case one forgets the password)
4. **Review the Audit Log regularly**
5. **Monitor the Health page**
6. **Keep LockWall updated**

### 🚨 Don't

- ❌ Don't run with `host: 0.0.0.0` without HTTPS
- ❌ Don't give Admin tokens when Viewer is enough
- ❌ Don't disable whitelist completely

---

## Before Production Checklist

Before enabling LockWall on a production server:

- [ ] Downloaded LockWall **only from official sources** (lockwall.app or GitHub Releases)
- [ ] **Verified the SHA-256 checksum** against the release page
- [ ] Changed the **default admin password**
- [ ] Added **admin, VPN and internal networks** to the whitelist
- [ ] Confirmed **Windows audit policy** is enabled (LockWall enables it automatically)
- [ ] Web UI is bound to **`127.0.0.1`** (or placed behind an HTTPS reverse proxy)
- [ ] Tested **Telegram or Email** notifications (if you use them)
- [ ] Selected a **GeoIP provider** or left GeoIP **Off**
- [ ] Backed up **`config.yaml`** and **`data\lockwall.db`**
- [ ] Confirmed an **alternative access method** to the server exists (in case of lockout)

---

## Privacy and Data Processing

LockWall is a **local-first** Windows Server security tool. By default it stores
operational security data **locally on the protected server**, including:

- attacker IP addresses;
- attempted usernames from Windows authentication events;
- timestamps and event source (RDP / OWA / SQL);
- block / unblock history and audit-log records;
- application logs and configuration values.

**LockWall does not store passwords.** The Windows Event Log does not expose attempted
passwords, and LockWall does not record them.

**External communication happens only when you enable a feature that needs it:**

- an **online GeoIP provider** (ip-api.com or ipinfo.io) — the attacker IP is sent to that provider;
- **Telegram** or **Email** notifications;
- any future webhooks / integrations.

If attacker IPs must never leave the server, use GeoIP mode **`Off`** or **`Local` (MaxMind)**.
You are responsible for ensuring your use of LockWall complies with the privacy, data-protection,
employment and logging laws that apply to your organization and jurisdiction.

---

## FAQ

### Q: How much resources does LockWall use?
**A:** Average 30-50 MB RAM, <1% CPU. Up to 100 MB during active attacks.

### Q: Can I use it on Windows Home?
**A:** Yes, but Windows Home isn't recommended for server tasks. LockWall works on all editions of Windows 10/11/Server 2016+.

### Q: What happens if LockWall crashes?
**A:** Service auto-restarts (Recovery configured in SCM). Block rules are stored in Windows Firewall — even if LockWall is down, IPs remain blocked.

### Q: Can I block entire countries?
**A:** Not directly via UI. You can use external scripts with GeoIP database + netsh.

### Q: Are attacker passwords stored?
**A:** **No.** LockWall only saves username and IP. Passwords aren't recorded — neither Windows nor LockWall does this.

### Q: How many IPs can be blocked simultaneously?
**A:** Windows Firewall limit ~10000 IPs per rule. With 6 rules → theoretically 60000 IPs. In practice rarely more than a few hundred.

### Q: Can LockWall protect HTTP/FTP?
**A:** Not in this version. Only RDP, OWA, SQL Server, and SSH. Future versions may extend support.

### Q: What if an IP keeps attacking?
**A:** Use Progressive Blocking (4th block = permanent), or manually **Make Permanent**. Consider blocking the entire subnet via CIDR.

### Q: How do I verify LockWall is actually blocking?
**A:**
1. Open **Blocked IPs** — should have entries
2. `netsh advfirewall firewall show rule name=LOCKWALL_BLOCK_7D` — should have IPs
3. Try RDP from blocked IP — should fail
4. **Health page** — Firewall sync should be OK

### Q: Can I integrate with SIEM (Splunk, ELK)?
**A:** API tokens and the current JSON endpoints can be used for basic automation
(see the **API Status** section above). A fully documented, versioned public REST API
(`/api/v1/…`) is planned for a future release.

### Q: Where is the database stored?
**A:** `C:\LockWall\data\lockwall.db` — SQLite. Can be backed up.

### Q: How to test without real attacks?
**A:** `lockwall.exe run --dry-run --web` — simulation mode (logs what it *would* block, without touching the firewall).

---

## Additional Resources

- **Website**: https://lockwall.app/
- **GitHub**: https://github.com/SMSerg2000/LockWall
- **Issues / feature requests**: https://github.com/SMSerg2000/LockWall/issues
- **Why LockWall is free**: [PHILOSOPHY.md](PHILOSOPHY.md)

---

**LockWall** — Protect your Windows Server like a real fortress! 🛡️
