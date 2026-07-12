# Changelog

All notable changes to **LockWall** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.1.0] — 2026-07-12

🔑 **SSH protection & bulletproof startup.**

### Added
- **SSH brute-force protection** (Windows OpenSSH Server) — LockWall watches the
  **OpenSSH/Operational** event log and auto-blocks IPs with repeated failed
  `sshd` logins. Attempts share thresholds with RDP/OWA/SQL — one attacker, one
  block, whichever door they knock on; progressive blocking and password-spray
  detection apply too. Off by default: **Settings → SSH Protection** (requires
  OpenSSH Server; service restart to apply).
- **`lockwall.exe test-notify`** — send a test Telegram/Email notification from
  the console to verify delivery without waiting for a real attack.

### Reliability
- The service now **survives early system boot**: engine initialization retries
  with backoff while Windows Firewall (BFE) is still starting, the service
  registers dependencies on BFE/EventLog, and install configures an SCM
  auto-restart recovery policy. No more silent startup deaths (Event 7034).
- **Web UI is isolated from protection** — a busy dashboard port no longer
  takes the blocking engine down; the web retries in the background.

### Fixed
- Telegram/SMTP notifications sent by the service could fail certificate
  verification (`CERTIFICATE_VERIFY_FAILED`) despite a valid certificate.
  TLS trust now uses the **native Windows certificate store**, with
  verification kept ON.

---

## [2.0.0] — 2026-06-28

🎉 **First public release.**

LockWall is now available to everyone as free proprietary freeware —
all three protocols, every feature, no license key, no seat limits.

### Protection
- **RDP, OWA & SQL Server** brute-force monitoring in a single Windows service
- **Password spray detection** — blocks IPs probing multiple usernames
- **Progressive blocking** — escalating duration for repeat offenders (1h → 24h → 7d → permanent)
- **Whitelist** with CIDR support, so you never block trusted networks
- **GeoIP** enrichment — opt-in by design (**off by default**, safest start).
  Pick a provider in Settings: **ipinfo.io Lite** (token, commercial-OK with
  attribution that LockWall shows automatically), **Local MaxMind** (offline,
  bring-your-own .mmdb), or **ip-api.com Free** (personal / non-commercial only)

### Web interface
- Dashboard, Blocked IPs, Analytics, Logs viewer, Health, Audit Log
- Multi-admin **User Management** with RBAC (Admin / Viewer)
- **Settings** — full configuration from the UI, no YAML editing required

### Integration
- **Telegram + Email** notifications on every block
- **CSV export** and **API tokens** (Bearer / X-API-Key)

### Installation
- **MSI installer** (`LockWall-Setup-x64.msi`) — presents the license, installs to
  `C:\LockWall`, and registers + starts the Windows service automatically
- Each release ships a **SHA-256 checksum** and a **VirusTotal** scan for verification

---

[2.1.0]: https://github.com/SMSerg2000/LockWall/releases/tag/v2.1.0
[2.0.0]: https://github.com/SMSerg2000/LockWall/releases/tag/v2.0.0
