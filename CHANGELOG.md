# Changelog

All notable changes to **LockWall** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.4.5] — 2026-08-22

🔥 **Windows Firewall awareness, hardened permissions, and a faster service.**

### Added
- **LockWall now tells you when Windows Firewall is switched off.** Until now it
  would detect attacks and create blocking rules that Windows silently never
  enforced — the dashboard filled up, notifications arrived, and the traffic kept
  flowing. The worst kind of broken protection is the kind that looks like it is
  working. The warning appears at service start (log), in a banner on **every**
  page of the web interface, on the new **Health** page, and on the installer's
  final screen — many people install through the MSI and never see a console.
- **One-click "Enable Windows Firewall"** on the Health page, or
  `lockwall.exe enable-firewall` on a server without a browser. Your RDP is
  protected first: LockWall reads the **real** RDP port from the registry (not
  just 3389) and creates an allow rule for TCP **and** UDP, inbound **and**
  outbound, *before* switching the firewall on. You also get a pre-flight list of
  everything currently listening, so you can see what would become unreachable.
- **Safety net against locking yourself out** — unless you confirm that access
  still works, the firewall switches back off within 10 minutes. Lost your
  session? Do nothing: the server reverts on its own. Configurable via
  `firewall.revert_timeout_minutes`. LockWall never enables the firewall on its
  own — that changes what every service on the machine can reach, so it stays
  your decision. Blocked attackers stay blocked throughout: Windows evaluates
  block rules before allow rules.
- **New commands for headless servers:** `lockwall.exe firewall-status` /
  `enable-firewall` / `confirm-firewall`.
- **The service description now carries the running version**
  (`services.msc → LockWall → Properties`), so you can tell which build a server
  is on without opening the web interface. Refreshed at every start, even if you
  updated by replacing files by hand.

### Security
- **The configuration and data folders are now restricted to SYSTEM and
  administrators.** They used to inherit permissions from the drive root that let
  **any** local user read `config.yaml` — the Telegram bot token, the SMTP
  password, and the key used to sign web sessions, which is enough to forge a
  cookie and enter the web interface as an administrator without a password.
- **The installation folder is now protected against modification.** The root of
  the system drive grants every authenticated user Modify permissions on
  everything created inside it, so any user of the machine could replace
  `lockwall.exe` and have the LockWall service run their code as SYSTEM at the
  next start. Write access is now limited to SYSTEM and administrators; reading
  stays open so logs remain available for diagnostics without administrator
  rights.
- Permissions an administrator granted to **specific accounts** are left
  untouched — the lockdown removes access for broad groups, not for people you
  trusted on purpose. Existing installations are corrected automatically on
  update or service start.

### Changed
- **LockWall no longer unpacks itself on every start.** It used to be a single
  self-extracting executable writing ~17 MB to a temporary folder each time it
  ran; the program files now sit next to the executable and load directly. The
  service starts noticeably faster and the spurious *"did not respond in a timely
  fashion"* warning (Event 7039) is gone.
- **Updates run unattended and ask nothing.** Blocked IPs stay blocked
  throughout, because the rules live in Windows Firewall rather than inside
  LockWall.

### Fixed
- **The web interface could start returning "Internal Server Error"** after the
  service had been running for a while. Windows cleans up temporary folders
  periodically — even while a service is running — and took the web interface's
  files with it. Protection kept working throughout; only the web interface was
  affected. With the unpacking gone, so is the cause.
- **The Health page reported the firewall check as "ok" while the firewall was
  off** — it only compared rule counts, which exist regardless of whether Windows
  enforces them.
- **The installer could freeze indefinitely with the progress bar full.** Windows
  consoles pause any program writing to them while text is being selected, so a
  single stray mouse click was enough — trivially easy over RDP. This also
  protects console runs, where the same click would have frozen the protection
  engine.

### Documentation
- The **API section** now documents what to do with a token, not just how to
  create one: every endpoint with its required role, error codes, and PowerShell
  examples rather than curl only. Note that blocking and unblocking IPs is not
  yet available through the API — that arrives with the versioned `/api/v1`.

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
