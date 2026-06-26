# Security Policy

## Reporting a Vulnerability

LockWall is an independently maintained project, and its security is taken seriously. If you discover a security vulnerability, please **do not** open a public GitHub issue.

### How to Report

Please report security vulnerabilities via **email**:

📧 **support@lockwall.app**

### What to Include

A good vulnerability report includes:

1. **Description** — what the vulnerability is
2. **Impact** — what an attacker could do
3. **Reproduction steps** — how to reliably trigger it
4. **Affected versions** — which LockWall versions are vulnerable
5. **Proof of concept** — code or commands demonstrating the issue (if applicable)
6. **Suggested fix** *(optional)* — how it could be addressed
7. **Your contact info** — email or PGP key for follow-up
8. **Disclosure preference** — credited or anonymous?

### What to Expect

We follow a **responsible disclosure** approach:

| Step | Timeline |
|------|----------|
| **Acknowledgement** | Within 48 hours |
| **Initial assessment** | Within 7 days |
| **Status updates** | Every 7 days |
| **Fix release** | Depending on severity (Critical: ASAP; High: 30 days; Medium: 60 days; Low: next release) |
| **Public disclosure** | After fix release + 90 days (or earlier with reporter consent) |

### Severity Classification

#### 🔴 Critical
- Remote code execution
- Authentication bypass leading to full system access
- SQL injection allowing data exfiltration
- Privilege escalation to admin without auth

#### 🟠 High
- Authentication bypass with limited access
- Persistent XSS in web UI
- Disclosure of password hashes / API tokens
- DoS that crashes the service
- Path traversal allowing arbitrary file read

#### 🟡 Medium
- Reflected XSS
- CSRF on sensitive endpoints
- Information disclosure (non-sensitive data)
- Denial of service requiring authentication
- Insecure default configurations

#### 🟢 Low
- Missing security headers
- Information disclosure of public information
- Issues requiring physical access

---

## Supported Versions

Security updates are provided for the latest version:

| Version | Supported |
|---------|-----------|
| 2.0.x | ✅ Supported |
| 1.x / pre-public builds | ❌ Not supported |

---

## Security Best Practices for Users

### Mandatory

1. ✅ **Change default password** immediately (forced on first login)
2. ✅ **Whitelist your trusted networks** before exposing the service
3. ✅ **Don't expose port 8880 externally** without an HTTPS reverse proxy
4. ✅ **Run as the Windows Service** under LocalSystem, not as a user

### Recommended

5. 🛡️ **HTTPS reverse proxy** (nginx, Caddy, IIS) for any external access
6. 🛡️ **Multiple admin accounts** (in case one loses access)
7. 🛡️ **Review the Audit Log regularly** for suspicious admin activity
8. 🛡️ **Monitor the Health page** for early problem detection
9. 🛡️ **Keep LockWall updated** — subscribe to GitHub releases
10. 🛡️ **Back up** `config.yaml` and `data\lockwall.db` regularly
11. 🛡️ **Use API tokens with least privilege** (Viewer role when possible)
12. 🛡️ **Rotate API tokens** periodically

### Don't

- ❌ Run LockWall with `host: 0.0.0.0` without HTTPS in front
- ❌ Disable the whitelist in production
- ❌ Share API tokens via insecure channels

---

## Cryptography

- **Password hashing**: `werkzeug.security` (scrypt by default)
- **API token storage**: SHA-256 hash of high-entropy random tokens; the raw token is never stored
- **Session keys**: signed via a persistent per-install secret
- **Random tokens**: 256 bits of entropy from a CSPRNG

---

## Out of Scope

The following are **not** considered security vulnerabilities for this policy:

- Issues requiring physical access to the server
- Issues in third-party dependencies (report to upstream)
- Social engineering or phishing
- Network-level (volumetric) DDoS that floods the host independently of authentication failures — this is outside LockWall's scope (it acts on failed-login events, not raw traffic volume)
- Missing best-practice headers when LockWall runs behind a reverse proxy that handles them

---

## Acknowledgement

Thank you for helping keep LockWall and its users safe! 🛡️

---

**LockWall** | support@lockwall.app
