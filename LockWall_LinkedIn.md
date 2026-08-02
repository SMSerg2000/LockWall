# LockWall — LinkedIn Content

**Language:** English · **Angle:** product announcement · **Date:** 2026-06-27
Two ready-to-publish pieces: a short feed post and a full LinkedIn Article.
Facts verified against the actual product source (7-day default block, Event 4625/18456,
netsh Windows Firewall, 0/61 VirusTotal, Windows 10 / Server 2016+).

> 💡 **LinkedIn tip:** the algorithm suppresses posts with external links in the body.
> Publish the post WITHOUT a link, then drop the link as the **first comment** (text below).

---

## 1) Short feed post

```
🛡️ Just shipped: LockWall — free brute-force protection for Windows Server.

If you run an internet-facing Windows box, you know the drill: thousands of failed
RDP logins a day, bots hammering away while you watch them scroll past in the Event Log.

LockWall watches with you — and shuts the door automatically.

It reads the Windows Security & Application logs, and when an IP crosses your
failed-login threshold, it gets blocked in Windows Firewall. RDP, Outlook Web Access
and SQL Server — one lightweight local service.

What it is:
✅ Free forever — every feature, no license key, no seat limits, no paid tier
✅ Runs 100% locally — no cloud, no agents; your data stays on your server
✅ Progressive blocking, password-spray detection, CIDR whitelist
✅ Web dashboard, Telegram/email alerts, audit log, analytics
✅ 0/61 on VirusTotal · SHA-256 published with every release

It's closed-source freeware — I kept the binary closed but made trust verifiable:
every release ships a SHA-256 hash and a VirusTotal scan.

I built it to solve a real problem on my own servers, and I'm sharing it because
protection like this shouldn't sit behind a paywall.

Free at home or at work. If it saves your server a headache, a ⭐ makes my day.

#WindowsServer #InfoSec #RDP #CyberSecurity #SysAdmin #Windows #BruteForce
```

### First comment (post the link here, not in the body)

```
Download & source 👉 github.com/SMSerg2000/LockWall
🌐 lockwall.app
```

---

## 2) Full LinkedIn Article

**Title:** LockWall: Free, Local Brute-Force Protection for Windows Server

**Alternative titles:**
- I built a free tool to stop Windows brute-force attacks — meet LockWall
- Free RDP / OWA / SQL brute-force protection for Windows, the local-first way

```
Spin up a Windows server with RDP exposed to the internet, and within hours the bots
find it. The Security Event Log fills with Event 4625 — failed logon, failed logon,
failed logon — from IPs all over the world, methodically trying admin / administrator
/ sa against your box. Most of us cope by moving RDP off port 3389, hiding behind a
VPN, or simply living with the noise.

I wanted something that just handled it. So I built LockWall.


WHAT IT IS

LockWall is a free Windows service that watches for failed logins and automatically
blocks the attacking IPs in Windows Firewall — across three of the most-attacked
Windows services: RDP, Outlook Web Access (OWA), and SQL Server.

It runs entirely on your server. No cloud, no agents, no account to create.


HOW IT WORKS (no magic, no kernel drivers)

1. It reads the Windows Event Log — the Security log (Event 4625 for RDP and OWA)
   and the Application log (Event 18456 for SQL Server).
2. It counts failed attempts per source IP within a time window.
3. When an IP crosses your threshold, LockWall adds it to a Windows Firewall block
   rule via netsh — grouped rules, not one rule per IP.
4. Blocks expire automatically (default: 7 days) — or escalate for repeat offenders.

It's a lightweight polling service: idle CPU sits near zero, and it stays light even
during sustained attacks.


WHAT'S INSIDE

• RDP, OWA and SQL Server protection in one service
• Progressive blocking — escalating duration for repeat offenders (1h → 24h → 7d → permanent)
• Password-spray detection — catches one IP probing many usernames
• Whitelist with CIDR support — never lock out your office or VPN
• A local web dashboard — blocks, analytics, audit log, health checks
• Telegram and email alerts (optional)
• Optional GeoIP enrichment — country/city/ISP for attackers, off by default
• Multi-admin with roles (Admin / Viewer) and CSV export


FREE — AND WHY

LockWall is free forever. Every protocol, every feature. No license key, no trial,
no "Pro" edition, no seat limits. Run it on one server or fifty, at home or at work.

I run it on my own production servers every day. Sharing it costs me nothing, and
real protection against a flood of brute-force attempts belongs in every admin's
hands — not behind a paywall.


CLOSED-SOURCE, BUT VERIFIABLE

LockWall is proprietary freeware — the binary is closed. For a tool that manages your
firewall, though, trust matters more than ever. So every release is verifiable:
a SHA-256 checksum and a VirusTotal scan ship with each build. The current release is
0/61 clean — zero vendor flags, even unsigned.


GETTING STARTED

1. Download the MSI from GitHub Releases and run it — the service installs and starts
   automatically.
2. Open the dashboard at http://127.0.0.1:8880 and set your admin password.
3. Whitelist your trusted networks (so you never lock yourself out).

That's it. Requirements: Windows 10 / Server 2016 or newer, 64-bit.


This is my first public release, and I'd genuinely love feedback — bug reports and
feature ideas are welcome on GitHub Issues.

⬇️  github.com/SMSerg2000/LockWall
🌐  lockwall.app

If LockWall saves your server some grief, a ⭐ on the repo would mean a lot.
```

---

## Publishing checklist

- [ ] Post the short version; add the link as the **first comment**.
- [ ] Publish the Article (Articles are great for your profile / portfolio).
- [ ] Pin the post to your profile featured section.
- [ ] Reply to early comments quickly (boosts reach in the first hour).
- [ ] Optional: a banner/screenshot image lifts engagement — but keep it honest
      (no fabricated metrics like the "42ms" we flagged on the site).

---

# Variant 2 — "Built with AI" story

**Angle:** personal journey (AI as accelerator, not autopilot) · **Timeline fact:** first
commit 2026-01-31, public release 2026-06-27 → ~5 months, 82 commits. Built in **Python**
(pywin32 + Flask + PyInstaller). AI framed as a tool; responsibility/verification stays the author's.

> Strategy: best published as a SEPARATE post a week or two after the product announcement —
> it gives the story its own stage and a second reason to post.

## 3) Short post — AI story

```
I'm 52. I'm an IT director, not a full-time developer.

Five months ago I started building a real security product. This week I shipped it
to the world — for free.

It's called LockWall: it watches the Windows Event Log and auto-blocks RDP, OWA and
SQL Server brute-force attackers in Windows Firewall. A problem I'd been fighting on
my own servers for years.

Here's the part I didn't expect to be writing: I built it with an AI pair-programmer
(Claude, mostly). Not "the AI wrote it for me" — more like a tireless senior engineer
on call who never sighs at a dumb question. In about a week I went from nervous to
writing real Python and using Git like I'd done it for years. Over five months that
became 80+ commits, an installer, a dashboard, and a public release.

But here's what mattered most: I verified everything. I tested it on production
servers for months. Every release ships a SHA-256 and a VirusTotal scan (0/61 clean).
AI was the accelerator — not the autopilot. The responsibility stays mine.

The lesson? The barrier to building real things has never been lower. If you've told
yourself "I'm not a developer" or "I'm too old to start" — you're not, and you're not.

LockWall is free, forever. Link in the comments. 👇

#BuildInPublic #AI #WindowsServer #InfoSec #NeverTooLate
```

### First comment

```
Download & source 👉 github.com/SMSerg2000/LockWall
🌐 lockwall.app
```

## 4) Full Article — AI story

**Title:** I'm 52, an IT Director — and I Just Shipped My First Product, Built with AI

```
I am not a software developer. I run IT for a logistics company. For most of my career,
"building a product" was something other people — younger people, full-time engineers —
did. This week I shipped one to the world. Here's the honest story of how, and what five
months of building alongside an AI actually taught me.


THE PROBLEM I COULDN'T LET GO

Every internet-facing Windows server I manage gets the same treatment: a relentless
drizzle of failed RDP logins, bots from everywhere trying admin / administrator / sa,
day and night. You see it in the Event Log and you learn to live with it. I got tired
of living with it.

I wanted a tool that just watched the logs and slammed the door on attackers
automatically. The ones I tried never quite fit. So — half as an experiment — I decided
to build my own.


WHAT AI ACTUALLY CHANGED

I'd written scripts before, but never a real, shippable product. This is where an AI
pair-programmer (Claude, mostly) changed everything — not by writing it "for me," but by
removing the friction that usually stops people like me before they start.

In about a week I went from nervous to genuinely comfortable — writing Python, using Git,
reading Windows APIs I'd never touched. Not because I'm gifted, but because I had a
patient senior engineer on call who never made me feel stupid for asking. Every "why
doesn't this work?" got a real answer; every dead end had a way out. Over five months,
that became 80+ commits, a Windows service, an MSI installer, and a web dashboard.


THE PART THAT MATTERS: I OWNED IT

Here's what I want to be clear about, because it's the difference between a toy and a
tool: AI was the accelerator, not the autopilot.

I read and understood what went into the product. I tested it on real production servers —
it's been blocking real attackers for months. And because it's a security tool that
touches your firewall, I made trust verifiable: every release ships a SHA-256 checksum and
a VirusTotal scan. The current build is 0/61 — zero vendor flags.

AI helped me build faster. The responsibility for what I shipped stayed mine. I think
that's the right way round.


WHAT IT BECAME: LOCKWALL

LockWall is a free Windows service that auto-blocks brute-force attackers across RDP,
Outlook Web Access and SQL Server. It reads the Windows Event Log, and when an IP crosses
your failed-login threshold, it's blocked in Windows Firewall. Progressive blocking,
password-spray detection, a CIDR whitelist, a local dashboard, Telegram/email alerts — all
running on your own server, no cloud.

It's free forever — every feature, no license key, no paid tier. I run it on my own
servers, and protection like this shouldn't sit behind a paywall.


THE LESSON

I'm 52. Five months ago I'd never shipped software. The barrier to building real things has
never been lower — not because AI does the work, but because it removes the fear of starting
and the cost of not knowing yet.

If you've been telling yourself you're "not technical enough" or "too old to start," I'd
gently push back. Pick the problem that annoys you most, and start. You might be surprised
what you ship in five months.

LockWall is on GitHub and at lockwall.app — free to use, at home or at work. If it saves
your server some grief, a ⭐ would make my day.

⬇️ github.com/SMSerg2000/LockWall
🌐 lockwall.app
```
