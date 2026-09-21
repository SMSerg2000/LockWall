# Changelog

All notable changes to **LockWall** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.6.6] — 2026-09-21

### Fixed
- **Commands other than the service itself now write to the log.** Only the
  service and three commands did; everything else wrote to a logger with nowhere
  to write. The costly one was the firewall safety net: when you enable Windows
  Firewall from LockWall, a scheduled task turns it back off unless you confirm
  your access still works — and that revert, running with nobody at the console,
  left no trace at all. The same applied to `enable-firewall`, `confirm-firewall`,
  `install`, `uninstall`, and to the startup message about certificate
  verification, whose failure quietly explains why notifications later stop
  arriving. If LockWall cannot write its log, it keeps running anyway.

## [2.6.5] — 2026-09-21

### Added
- **The About page marks a server that is on the beta channel**, next to the
  version. Beta servers receive releases before everyone else; until now that was
  visible only in *Settings*. Releases keep one version number across channels:
  what reaches stable is the same file, byte for byte, that was tried on beta.

## [2.6.4] — 2026-09-21

### Fixed
- **SSH scanners that present a username and drop the connection before sending
  any password were invisible.** sshd records such a probe as `Invalid user X from
  IP port N` followed by `Connection reset by invalid user … [preauth]` — and never
  writes the *Failed password* line, the only line LockWall had counted since SSH
  protection appeared in 2.1.0. A bot cycling through `root`, `admin` and `user1`
  could knock all day without a single block. Unknown usernames now count as
  attempts, as fail2ban treats them by default; wrong passwords sent for an unknown
  user count as well. Connection close/reset lines are still ignored — they belong
  to a probe that has already been counted. Reported by a LockWall user; thank you.

## [2.6.3] — 2026-09-21

### Fixed
- **The *Updates* switches in Settings now take effect immediately**, as the page
  says. In 2.6.2 the running service kept its previous settings until it was
  restarted: you could enable update checking, press *Save*, see the confirmation
  — and the service still would not be checking. *Check now* on the About page
  worked, which made it easy to miss. Coming from 2.6.2 with updates that never
  seemed to start: restart the service once, or install this release by hand.
- **No warning in `logs\update.log` when an update succeeds.** The installer
  starts the service itself; LockWall then tried to start it again and logged the
  resulting "already running" as a warning. The outcome was always correct.

### Added
- The service log states the update settings at startup — channel, how often it
  checks, and whether installing is automatic. Handy for reading the state of many
  servers from their logs instead of opening each web interface.

## [2.6.2] — 2026-09-20

🔄 **Automatic updates — signed, verified, rolled back if needed.**

### Added
- **Automatic updates, off by default.** LockWall makes no outbound requests
  unless you enable checking in *Settings → Updates*. When enabled, the service
  fetches two small files from the release page every 10 minutes — the manifest
  and its signature — and sends nothing about your server. With *Install updates
  automatically* it installs new releases on its own; otherwise you get a banner,
  an *Install now* button on the About page and `lockwall.exe update`.
- **Every update is signed.** The manifest is verified with a key built into
  LockWall *before* it is parsed; the installer must match the size and SHA-256
  from the signed manifest; HTTPS is enforced through redirects; an older version
  is never installed. A backup key is built in for rotation.
- **Installs when quiet, verifies, rolls back.** Waits for two minutes without new
  blocks (at most half an hour), copies the program aside, installs silently from a
  one-off scheduled task, restarts the service and checks that the new version is
  actually running — otherwise the previous files are put back. Outcome to the
  log, the About page and Telegram/Email. Blocked IPs stay blocked throughout.
- **Waves:** a `beta` channel gets releases first, then a growing share of `stable`
  servers; membership is computed locally, no server names travel anywhere.
- New commands `check-update` and `update`; Updates card in Settings; update panel
  on the About page.
- **Windows Firewall is watched while LockWall runs.** Switched off → ERROR in the
  log and an alert to Telegram/Email within five minutes, hourly reminders, and a
  message when it is back on. The startup warning reaches Telegram/Email too.

### Changed
- Blocked IPs are reconciled with Windows Firewall every few minutes, not only
  when someone opens the Blocked IPs page.
- Faster handling of large attacks: in-memory whitelist, per-address firewall
  updates no longer hold up other addresses, dashboard statistics no longer pause
  detection.
- Python runtime updated to 3.14.

### Upgrading
Install the MSI over any 2.x version; settings, database and blocked IPs are
kept. This is the last manual update if you enable automatic ones afterwards.

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
