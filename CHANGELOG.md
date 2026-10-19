# Changelog

All notable changes to **LockWall** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.9.3] — 2026-10-19

Full notes: [release v2.9.3](https://github.com/SMSerg2000/LockWall/releases/tag/v2.9.3). Versions 2.8.5,
2.9.0, 2.9.1 and 2.9.2 went to our beta servers only and were never published as stable; their changes
are listed here.

### Added
- **Alert on a successful logon after failed attempts.** The worst outcome of brute force is a guessed
  password. LockWall now watches successful network logons as well (Security event 4624 for RDP and
  OWA, `Accepted` lines from OpenSSH). When an address logs on successfully after 3 or more failed
  logons within an hour, you get an alert 🔓 with the account, the address, the service and the user
  names that were tried. Nothing is blocked automatically. Threshold and window: **Settings → Detection**.
- **Subnet blocking.** When 3 different addresses of one /24 are blocked within an hour, the whole /24
  is blocked for a day; the alert says *DISTRIBUTED* with 🌐. Public IPv4 only; never a subnet that
  overlaps the whitelist or holds this server's own public address.
- **Weekly summary.** Every Monday at 09:00 server time: the week's blocks by service, failed logons,
  the busiest addresses, and the state of the firewall, the Security log and logon auditing 📊.
  **Settings → General → Weekly summary** turns it off.
- **Blocked IPs keeps your place.** After Make permanent, Make temporary or Unblock the page returns to
  where you were and keeps the sort order. The **Actions** header sorts by block type: temporary
  blocks first, click again for permanent first.

### Security
- **Secrets in config.yaml are encrypted.** The Telegram bot token, the SMTP password and the
  ipinfo.io token are stored encrypted with a key of the server (Windows DPAPI), so a copy of
  `config.yaml` is useless on any other computer. LockWall encrypts them itself once the service has
  been running for 10 minutes. Settings no longer shows them: leave a field empty to keep the saved
  value. An administrator of the server can still recover them; the protection is against copies of
  the file.

### Changed
- **The whitelist you see is the whitelist that works.** The protection checked a copy of the whitelist
  in its database that was only ever added to, so an address removed in Settings stayed unblockable.
  The database now follows `config.yaml`. On the first start, entries that only the database had are
  added to `config.yaml` and shown in Settings, so the upgrade never takes protection away.
- **Settings take effect on Save**: detection thresholds, subnet blocking, the server label and the
  whitelist no longer wait for a restart. `config.yaml` is written atomically.
- **The server's public address is looked up only while Telegram or Email is on.** 2.8.4 asked
  `api.ipify.org` on every server, including servers that send no notifications.
- **A failed update is not retried in a loop.** A version whose installation stopped the service and
  failed is retried automatically only after Windows restarts or after 24 hours; **Install now** still
  tries at once.

### Fixed
- Under a heavy attack the Security and Application log readers re-read their whole backlog every
  second; unprocessed events are now kept between cycles.
- IPv6: blocked IPv6 addresses no longer drift apart from the database, the web forms accept IPv6
  addresses and subnets, and local IPv6 addresses are never sent to an online GeoIP service.
- An address unblocked in the database but still in the firewall rule is taken back into the database
  by the next reconciliation. A block list longer than a netsh command line (about 1,900 addresses) is
  written through the Windows Firewall COM interface. The local MaxMind database is opened once.
- Cancelling the *Unblock?* confirmation no longer leaves the button disabled with a spinner.

## [2.8.4] — 2026-10-17

Full notes: [release v2.8.4](https://github.com/SMSerg2000/LockWall/releases/tag/v2.8.4). Versions 2.8.2
and 2.8.3 were internal builds and were never published; their changes are listed here.

### Added
- **A firewall that is on but lets everything in is reported, and can be fixed.** Every Windows
  Firewall profile has a default inbound action. When it is Allow, every inbound connection that no
  rule explicitly blocks gets in, with the firewall switched on. On one of our servers the Private and
  Public profiles were set that way: Windows Security showed it in red, LockWall showed green, because
  it only checked whether the firewall was on. LockWall's own blocks kept working (block rules win),
  but everything else listening on the server was reachable. LockWall now reads the default inbound
  action of every profile. When the active profile allows inbound connections by default, it says so
  in the log, on every dashboard page, on the Health page and in `lockwall.exe firewall-status`, and
  sends one alert; a switch while it runs is noticed within ten seconds. **Block inbound by default**
  on the Health page, or `lockwall.exe block-inbound`, shows which listening ports have no allow rule
  and will close, creates an allow rule for the real RDP port first, arms the 10-minute automatic
  revert, then switches inbound to Block and leaves outbound as it was. LockWall never does this on
  its own, and leaves a setting that comes from Group Policy to the GPO.
- **Alerts you can search by icon.** Every Telegram message starts with an icon for its type:
  ⛔ an attacker was blocked, 🧱 Windows Firewall, 🦠 Microsoft Defender, 📜 the Security log or logon
  auditing, 💾 SQL Server without SQL protection, 🔧 LockWall itself, 📦 automatic updates, 🧪 a test
  message. Each icon belongs to one type only. Service alerts keep a second icon for how serious they
  are: 🚨 needs action, ⚠️ a warning, ✅ back to normal.
- **The server's public address in every message.** `Server: TERMINAL (203.0.113.5)` instead of
  `Server: TERMINAL`, in Telegram, in email and in the email subject. LockWall looks the address up at
  `api.ipify.org` over HTTPS once an hour, with `checkip.amazonaws.com` as a fallback. This is a new
  outbound request, made whether or not notifications are on; only the request itself leaves the
  server. **Settings → General → Public IP**: empty for automatic detection, an address to use as
  typed, `off` to leave it out and stop the lookup. A message never waits for the lookup.

### Changed
- A password spray inside a block message is marked 🎯 instead of ⚠️, so searching for warnings no
  longer finds blocks.
- The User Guide says that a Telegram group works as well as a private chat (use the group's Chat ID
  with its minus sign), and its privacy section lists the public address lookup.

## [2.8.1] — 2026-10-12

Full notes: [release v2.8.1](https://github.com/SMSerg2000/LockWall/releases/tag/v2.8.1).

### Added
- **Slow brute force is blocked.** The regular rule blocks an address after 5 failed logons
  within 10 minutes, so an attacker who keeps a slower pace never reached it: the day 2.8.0 came
  out, one address tried the `sa` password on one of our SQL Servers once every 36 minutes, 17
  times a day. LockWall now also blocks an address after 20 failed logons within 24 hours, counting
  RDP, OWA, SQL and SSH together. The block lasts as long as a regular brute-force block, the reason
  reads *Slow brute force*, the Blocked IPs page marks it **SLOW** and the alert says *SLOW BRUTE
  FORCE*. On by default; tune it in Settings or in `detection.slow_bruteforce` in `config.yaml`.

## [2.8.0] — 2026-10-12

A serious fix, released straight to all servers. Full notes:
[release v2.8.0](https://github.com/SMSerg2000/LockWall/releases/tag/v2.8.0).

### Fixed
- **SQL protection now works with SQL Server in any language.** SQL Server writes the address
  of a failed logon (event 18456) as `[CLIENT: address]`, but translates the word *CLIENT* into
  the language of the instance. LockWall looked for the English word only, so on a SQL Server
  installed in another language — Russian, in our case — SQL protection read every failed logon
  and counted none of them: on one of our servers 7,342 password guesses against `sa` in a single
  day went unblocked. LockWall now reads the address whatever the word in front of it is, checks
  that it is a real IPv4 or IPv6 address, and still leaves local connections alone.

### Added
- **A failed SQL logon without an address LockWall can read is reported** as a warning in the
  log, at most once an hour, instead of being skipped silently.

## [2.7.4] — 2026-10-11

Contains 2.7.3, which was installed by hand on seven of our own servers and never published, plus
two fixes from its first evening in the field. Everything here comes from one incident: a server
where Defender had detected a hacking tool inside SQL Server three days before the server was
encrypted, and nobody was reading the Defender log. Full notes:
[release v2.7.4](https://github.com/SMSerg2000/LockWall/releases/tag/v2.7.4).

### Added
- **Microsoft Defender detections and protection switches go to Telegram and Email.** LockWall
  reads the Defender event log once a minute and reports what Defender finds (the threat and its
  severity, where it was found, the process and user it ran under, what Defender did about it, and
  whether it ran inside SQL Server), real-time protection switched off (a red banner on every page,
  an hourly reminder and a message when it is back), a setting that weakens Defender, a change
  reverted by Tamper Protection, an emptied Defender log and the Defender service not running.
  Four or more detections within a minute come as one digest; after ten alerts in an hour the rest
  go to the log only. The new `defender` section in `config.yaml` is on by default and harmless
  next to another antivirus; there is a switch in Settings and `lockwall.exe defender-status`.
- **A firewall switched off is noticed within ten seconds** instead of on the next five-minute
  check — the same alarm as before, just in time.
- **The updater waits two minutes before declaring success** and requires the protection engine
  to report that it is active. If the service dies or the engine does not come up, the previous
  version is restored and the alert says what happened.
- **An unreadable database no longer leaves the service running with nothing inside.** The file
  is set aside as `data\lockwall.db.unreadable-<date>`, LockWall starts with an empty database,
  reads the active blocks back from the Windows Firewall rules and sends an alert that says what
  was lost (history, web users and API tokens; the admin password is back to the default). If the
  engine still cannot start, an alert goes out on the third failed attempt and when LockWall gives
  up, at most once an hour.
- **SQL Server without SQL protection is pointed out** — a line in the log, a banner on every page
  and one alert when a running SQL Server service is found while SQL protection is off.

### Changed
- A Defender setting that keeps flipping by itself no longer floods the chat: it is reported twice
  within two hours, then muted (one message says so) until it has stayed unchanged for a day.
- A Tamper Protection event that reverted a change which changed nothing (`0 -> 0`) goes to the
  log instead of Telegram.

## [2.7.2] — 2026-10-09

The first stable release since 2.6.8. It includes 2.6.9, 2.6.10, 2.7.0 and 2.7.1, which were
tested on the beta channel. Full notes: [release v2.7.2](https://github.com/SMSerg2000/LockWall/releases/tag/v2.7.2).

### Added
- **LockWall warns when Windows stops writing the Security log.** If the log is full and set to
  *Do not overwrite events*, Windows silently stops recording failed logons, and LockWall went
  blind while still showing *Running*. It now checks the log at start and every five minutes:
  a red banner, a Telegram or email alert, hourly reminders and a message when the log recovers.
  *Fix the Security log* on the Health page (or `lockwall.exe fix-security-log`) saves a copy and
  switches the log to *Overwrite events as needed*; `lockwall.exe security-log-status` checks it
  from the console.
- **The audit policy is watched while the service runs**, not only at start. If failed-logon
  auditing is switched off, LockWall turns it back on (`audit.auto_enable`, on by default) and
  tells you, or raises an alert.

### Fixed
- **Failed logons in the first minutes after a Windows boot are counted.** The service starts two
  to three minutes after Windows. On the first start after a boot LockWall now reads the event
  logs back to the boot, but no further than `time_window_minutes`; a restart within the same
  boot reads nothing twice.
- **Health page cards** *Audit Policy*, *Security Log* and *System* show real checks. Since 1.6.0
  they said *Checking...* forever.

### Security
- **Email checks the SMTP server certificate.** STARTTLS accepted any certificate; port 465 now
  uses SMTP over SSL. `notifications.email.insecure_ssl: true` is the explicit opt-out.
- **A tighter update path.** The updater's working folder is limited to SYSTEM and
  Administrators; the installer is checked against the signed manifest once more right before it
  runs; a bad signature or a mismatching installer raises an alert instead of a log line.
- **OpenSSL 3.5.7** (Python 3.14.7) instead of OpenSSL 3.0, whose support ended on
  7 September 2026.

### Changed
- **The log is in English.** All 165 messages that were still in Russian are translated, and the
  `config.yaml` created at install is in English too, with an `update` section (off).
- **The installer's error screen says what to do:** restart Windows, run setup again, and check
  that the LockWall service is running.

## [2.6.8] — 2026-09-21

### Added
- **`lockwall.exe` carries publisher, product, description and version in its
  file properties.** Those fields were empty before: Windows showed an unnamed
  executable. *Properties → Details* now shows `Serhii Smoktii` and `LockWall`,
  the same publisher the installer carries. An executable with no identity is one
  of the things antivirus machine-learning models hold against a file, and the
  effect is measurable: 2.6.7 was flagged by seven engines on VirusTotal
  including Microsoft Defender, this build by four — Defender, Elastic and
  Skyhigh are gone. Not a substitute for a code-signing certificate, but if you were holding
  off because Defender complained, this is the build to take.

## [2.6.7] — 2026-09-21

### Added
- **The About page says whether the installed version is a pre-release or the
  stable one**, next to the version number. This is about the file on disk, not
  about the update channel in Settings: a pre-release is simply a version newer
  than the one currently in the stable channel. Once it is released to everybody
  the label becomes *stable* by itself — nothing to reinstall, it is the same
  file. If LockWall cannot tell what is in the stable channel (checks off, no
  network, own mirror), no label is shown: saying "stable" when it might not be
  true would be worse than saying nothing.

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
