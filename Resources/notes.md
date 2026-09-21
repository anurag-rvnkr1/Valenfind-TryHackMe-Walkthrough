# 📝 Valenfind — Penetration Testing Notes

> Technical field notes recorded during the assessment of the **TryHackMe Valenfind** web application challenge. These notes summarize reconnaissance, enumeration, payload testing, observations, exploitation methodology, and remediation recommendations in a concise format suitable for future reference and portfolio documentation.

---

## Lab Information

| Item                  | Value                      |
| --------------------- | -------------------------- |
| Platform              | TryHackMe                  |
| Room                  | Valenfind                  |
| Category              | Web Application Security   |
| Difficulty            | Easy                       |
| Operating System      | Linux                      |
| Web Framework         | Python Flask               |
| Database              | SQLite                     |
| Primary Vulnerability | Local File Inclusion (LFI) |

---

# Assessment Workflow

```text
Reconnaissance
      │
      ▼
Enumeration
      │
      ▼
Application Mapping
      │
      ▼
Input Testing
      │
      ▼
LFI Confirmation
      │
      ▼
Source Code Analysis
      │
      ▼
Sensitive Secret Discovery
      │
      ▼
Database Inspection
      │
      ▼
Security Remediation
```

---

# Phase 1 — Reconnaissance Notes

## Objective

Identify exposed services and technologies running on the target host.

### Tool Used

* Nmap

### Command Used

```bash
nmap -sC -sV <TARGET-IP>
```

### Why This Scan?

| Option | Purpose                      |
| ------ | ---------------------------- |
| `-sC`  | Execute default NSE scripts. |
| `-sV`  | Detect service versions.     |

### Observations

* Web application exposed.
* HTTP service discovered.
* Service fingerprint suggested **Python Flask** development application.
* Only a minimal attack surface was initially exposed.

### Evidence

* HTTP service accessible through browser.
* Port identified during enumeration.

---

# Phase 2 — Directory Enumeration Notes

## Objective

Discover hidden directories, endpoints, and application resources.

### Tool Used

* Gobuster

### Command Used

```bash
gobuster dir \
-u http://<TARGET-IP>:5000/ \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
-t 80
```

### Why Gobuster?

* Identify hidden content.
* Discover API routes.
* Find administrative resources.

### Observations

Potential directories and application endpoints became visible during enumeration.

### Takeaway

Always enumerate before exploiting.

---

# Phase 3 — Application Mapping Notes

## Objective

Understand application functionality before testing inputs.

### Manual Actions

* Register account.
* Authenticate.
* Browse dashboard.
* Visit user profiles.
* Edit profile.
* Inspect developer tools.

### Browser Tools Used

* Firefox Developer Tools
* Chromium DevTools

### Interesting Finding

A request responsible for loading profile themes accepted a user-controlled parameter.

### Security Observation

User-controlled file-loading functionality deserves immediate inspection.

---

# Phase 4 — HTTP Request Inspection

## Objective

Identify controllable parameters.

### Interesting Endpoint

```http
GET /api/fetch_layout
```

### Parameter Observed

```text
layout=<value>
```

### Initial Behavior

* Loads HTML layout.
* Response rendered dynamically.
* No visible validation performed client-side.

### Security Hypothesis

Possible server-side file loading.

---

# Phase 5 — Local File Inclusion Testing

## Objective

Determine whether directory traversal is possible.

### Initial Payload

```text
../../../../etc/passwd
```

### Test Purpose

Attempt to access a known Linux file.

### Successful Indicators

* File contents returned.
* Application accepted traversal payload.
* Arbitrary file read confirmed.

### Vulnerability Confirmed

**Local File Inclusion (LFI)**

### Security Impact

* Read internal files.
* Enumerate filesystem.
* Access application source code.
* Expose secrets.

---

# Useful LFI Payload Notes

## Linux Files Worth Testing

```text
/etc/passwd
/etc/hosts
/etc/os-release
/proc/self/cmdline
/proc/self/environ
```

### Application Files

```text
app.py
config.py
settings.py
.env
database configuration files
```

### Why These Files?

| File                 | Reason                    |
| -------------------- | ------------------------- |
| `/etc/passwd`        | LFI validation.           |
| `/proc/self/cmdline` | Running application path. |
| Application source   | Review routing logic.     |
| `.env`               | Environment secrets.      |

---

# Phase 6 — Source Code Review Notes

## Objective

Review application logic exposed through LFI.

### Items Searched

* Flask routes.
* Authentication logic.
* Secrets.
* API endpoints.
* Database path.
* File handling.

### High-Value Findings

* Administrative endpoint existed.
* Secret stored directly inside source code.
* SQLite database location identified.

### Security Lessons

Never hardcode secrets inside source files.

---

# Source Code Review Checklist

* [x] Hidden endpoints discovered.
* [x] API authentication mechanism identified.
* [x] Database location identified.
* [x] File loading implementation reviewed.
* [x] Secret management weakness identified.

---

# Phase 7 — Endpoint Analysis Notes

## Objective

Understand administrative functionality.

### Administrative Endpoint

```text
/api/admin/...
```

### Authentication Method

Custom HTTP header authentication.

### Observation

Application trusted a hardcoded administrative token.

### Security Risk

* Secret disclosure.
* Broken access control.
* Privilege escalation opportunity.

---

# Phase 8 — Database Inspection Notes

## Objective

Inspect exposed SQLite database.

### Tool Used

SQLite3

### Commands Used

```bash
sqlite3 cupid.db
```

### SQLite Commands

```sql
.tables
.schema users
SELECT * FROM users;
```

### Data Reviewed

* User accounts.
* Profile information.
* Metadata.
* Administrative records.

> Sensitive challenge values have been removed from this repository.

---

# SQLite Investigation Checklist

| Task                      | Status |
| ------------------------- | ------ |
| Open database             | ✅      |
| Enumerate tables          | ✅      |
| Review schema             | ✅      |
| Inspect records           | ✅      |
| Identify sensitive fields | ✅      |

---

# Evidence Collection Notes

## Screenshots Captured

| Stage                  | Screenshot Folder              |
| ---------------------- | ------------------------------ |
| Nmap                   | `docs/assets/recon/`           |
| Gobuster               | `docs/assets/recon/`           |
| Homepage               | `docs/assets/application/`     |
| Registration           | `docs/assets/application/`     |
| Login                  | `docs/assets/application/`     |
| Dashboard              | `docs/assets/application/`     |
| LFI Request            | `docs/assets/lfi/`             |
| `/etc/passwd` Response | `docs/assets/lfi/`             |
| Source Code            | `docs/assets/source-analysis/` |
| Admin Endpoint         | `docs/assets/source-analysis/` |
| SQLite Database        | `docs/assets/database/`        |

---

# HTTP Testing Notes

## Request Inspection Goals

* Query parameters.
* Headers.
* Cookies.
* Session behavior.
* Dynamic responses.

### Tools Suitable for Testing

* Browser Developer Tools.
* Burp Suite Community Edition.
* curl.
* Postman.

---

# Indicators of Vulnerability

## Potential LFI Indicators

* File names accepted as parameters.
* HTML fragments loaded dynamically.
* Relative path traversal succeeds.
* Unexpected filesystem responses returned.

### Warning Signs

* Unsanitized file input.
* Missing allowlist validation.
* Direct filesystem reads.

---

# Security Weakness Summary

| Weakness                         | Risk                               |
| -------------------------------- | ---------------------------------- |
| Local File Inclusion             | Arbitrary file disclosure.         |
| Directory Traversal              | Access outside intended directory. |
| Hardcoded Secrets                | Credential exposure.               |
| Sensitive Source Exposure        | Internal application disclosure.   |
| Insecure Administrative Endpoint | Unauthorized resource access.      |

---

# OWASP Mapping Notes

| OWASP Category                           | Relation                                |
| ---------------------------------------- | --------------------------------------- |
| Broken Access Control                    | Administrative endpoint trust model.    |
| Cryptographic Failures                   | Secret exposure through source code.    |
| Security Misconfiguration                | Unsafe file loading implementation.     |
| Vulnerable Components                    | Flask application configuration review. |
| Identification & Authentication Failures | Weak secret management.                 |

---

# Developer Remediation Notes

## Local File Inclusion

### Recommended Fixes

* Validate filenames.
* Allowlist templates.
* Reject traversal characters.
* Normalize file paths.

---

## Secret Management

### Replace Hardcoded Secrets With

* Environment variables.
* Secret management services.
* Configuration outside source control.

---

## Flask Recommendations

* Disable debug mode.
* Restrict sensitive routes.
* Validate user input.
* Handle file access securely.

---

## Database Protection

* Store databases outside web-accessible paths.
* Restrict download functionality.
* Apply authorization checks.
* Encrypt sensitive information where appropriate.

---

# Commands Reference

## Reconnaissance

```bash
nmap -sC -sV <TARGET-IP>
```

## Enumeration

```bash
gobuster dir -u http://<TARGET-IP>:5000 \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

## Database Inspection

```bash
sqlite3 cupid.db
.tables
.schema users
SELECT * FROM users;
.quit
```

---

# Payload Reference

## Directory Traversal

```text
../../../../etc/passwd
../../../../proc/self/cmdline
```

### Purpose

Validate file disclosure through LFI.

> Challenge-specific payloads have been generalized where appropriate.

---

# Lessons Learned

## Technical Lessons

* Always enumerate application functionality before exploiting inputs.
* Dynamic file-loading endpoints should be treated as high-risk.
* Source code exposure frequently leads to additional vulnerabilities.
* Hardcoded secrets significantly increase application impact after compromise.
* SQLite databases often contain sensitive operational information.

---

# Blue Team Perspective

## Detection Opportunities

* Monitor repeated traversal sequences.
* Detect requests containing `../`.
* Alert on unexpected file access.
* Monitor sensitive endpoint access attempts.
* Log abnormal download behavior.

---

# Red Team Perspective

## Skills Practiced

* Web reconnaissance.
* Directory enumeration.
* HTTP request analysis.
* LFI validation.
* Source code review.
* SQLite inspection.
* Vulnerability chaining.

---

# Portfolio Notes

This file accompanies the complete walkthrough documentation included in this repository.

Related documents:

* `README.md`
* `Documentation/Valenfind_Documentation.md`
* `Documentation/Valenfind_Documentation.docx`
* `docs/index.md`
* `docs/methodology.md`
* `docs/exploitation.md`
* `docs/remediation.md`

---

## Responsible Disclosure

This repository intentionally omits:

* TryHackMe flags.
* Administrative tokens.
* Sensitive credentials.
* Complete challenge answers.

Placeholders such as `<REDACTED_SECRET>` are used throughout the documentation to preserve the educational value of the walkthrough while respecting the challenge.
