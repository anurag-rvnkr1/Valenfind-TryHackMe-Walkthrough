# 💘 Valenfind — TryHackMe CTF Walkthrough

<div align="center">

![Valenfind Banner](assets/room-banner.png)

# Valenfind — TryHackMe Walkthrough

**A Professional Capture-the-Flag (CTF) Documentation & Security Analysis**

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Valenfind-red?style=for-the-badge\&logo=tryhackme)](https://tryhackme.com/)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-success?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Web%20Security-blue?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-LFI%20%7C%20Source%20Code%20Analysis-purple?style=for-the-badge)
![Portfolio](https://img.shields.io/badge/Portfolio-Cybersecurity-111827?style=for-the-badge)

*A complete security walkthrough documenting the methodology, reconnaissance, vulnerability discovery, exploitation process, and remediation of the **Valenfind** TryHackMe room.*

</div>

---

## 📌 About This Walkthrough

This GitHub Pages site documents my complete solution to the **Valenfind** Capture-the-Flag room from **TryHackMe**. The walkthrough follows a realistic penetration testing workflow, beginning with reconnaissance and ending with vulnerability remediation.

Unlike a simple write-up, this documentation explains **why each action was performed**, **what vulnerability was identified**, and **how the findings map to secure software development practices**.

> **Portfolio Goal**
>
> Demonstrate practical skills in web application security, vulnerability analysis, Linux enumeration, Flask application review, SQLite analysis, and secure coding remediation.

---

# 🎯 Learning Objectives

After completing this room, I was able to:

* Perform reconnaissance against a target web application.
* Enumerate exposed directories and application endpoints.
* Identify a **Local File Inclusion (LFI)** vulnerability.
* Read sensitive files from a Linux server through directory traversal.
* Analyze Flask source code to identify insecure development practices.
* Discover a hardcoded administrative secret inside application code.
* Access a restricted administrative API endpoint.
* Analyze an exported SQLite database.
* Explain the complete attack chain and recommend secure mitigations.

---

# 🧭 Walkthrough Navigation

| Phase        | Description                                |
| ------------ | ------------------------------------------ |
| **Phase 1**  | Reconnaissance with Nmap and Gobuster      |
| **Phase 2**  | Application Exploration and Authentication |
| **Phase 3**  | Network Traffic Inspection                 |
| **Phase 4**  | Local File Inclusion Discovery             |
| **Phase 5**  | Source Code Review                         |
| **Phase 6**  | Administrative Endpoint Analysis           |
| **Phase 7**  | SQLite Database Investigation              |
| **Phase 8**  | Security Findings & Attack Chain           |
| **Phase 9**  | Vulnerability Remediation                  |
| **Phase 10** | Key Takeaways                              |

---

# 🏠 Lab Overview

| Information               | Value                                                                     |
| ------------------------- | ------------------------------------------------------------------------- |
| **Platform**              | TryHackMe                                                                 |
| **Room**                  | Valenfind                                                                 |
| **Category**              | Web Application Security                                                  |
| **Difficulty**            | Easy                                                                      |
| **Primary Vulnerability** | Local File Inclusion (LFI)                                                |
| **Framework**             | Python Flask                                                              |
| **Database**              | SQLite                                                                    |
| **Operating System**      | Linux                                                                     |
| **Skills Demonstrated**   | Enumeration, LFI, Source Code Analysis, API Testing, SQLite Investigation |

---

# 🧱 Attack Surface Overview

The Valenfind application is a Python Flask web application running on **port 5000**.

During enumeration, the following attack surface was identified:

* Authentication pages.
* User dashboard.
* User profile pages.
* Dynamic theme loading endpoint.
* Hidden administrative API.
* SQLite backend database.

---

## Attack Path Summary

![Attack Flow Diagram](assets/remediation/attack-flow-diagram.png)

<table><tr><td>

### Kill Chain

1. Reconnaissance
2. Directory Enumeration
3. Network Request Inspection
4. Local File Inclusion
5. Source Code Disclosure
6. Secret Discovery
7. Administrative API Access
8. Database Export
9. Sensitive Information Disclosure

</td></tr></table>

---

# ⚔️ Phase 1 — Reconnaissance

The assessment began with identifying exposed services and web application endpoints.

## Nmap Enumeration

![Nmap Scan](assets/recon/nmap-scan.png)

**Objective**

Identify running services, versions, and entry points.

**Outcome**

* Port **5000** discovered.
* Flask web application identified.
* Initial attack surface confirmed.

---

## Gobuster Directory Enumeration

![Gobuster Enumeration](assets/recon/gobuster-enumeration.png)

**Objective**

Discover hidden directories and application resources.

**Outcome**

* Authentication endpoints discovered.
* API routes identified.
* Additional attack surface exposed.

---

# 🌐 Phase 2 — Application Exploration

The application was explored as a legitimate user before performing security testing.

## Homepage

![Homepage](assets/application/homepage.png)

Initial landing page for the dating application.

---

## User Registration

![Register](assets/application/register-page.png)

Creating a normal user account enables authenticated testing.

---

## Complete Profile

![Complete Profile](assets/application/complete-profile.png)

Profile completion introduces additional user-controlled input fields.

---

## Login Page

![Login](assets/application/login-page.png)

Authentication provides access to internal functionality.

---

## User Dashboard

![Dashboard](assets/application/dashboard.png)

Main authenticated dashboard displaying user profiles.

---

## Profile Page

![Profile](assets/application/profile-page.png)

The profile page contains the **Profile Theme** selector that later becomes the attack vector.

---

# 🌍 Phase 3 — Network Traffic Analysis

The browser's Developer Tools were used to inspect client-server communication.

## Network Inspector

![Network Inspector](assets/application/network-inspector.png)

### Key Finding

A request was made to:

```http
/api/fetch_layout?layout=theme_classic.html
```

This user-controlled parameter became the primary attack surface.

> **Security Observation**
>
> Any endpoint accepting filenames or paths should immediately be evaluated for path traversal or Local File Inclusion vulnerabilities.

---

# 📂 Phase 4 — Local File Inclusion (LFI)

The `layout` parameter was tested using directory traversal payloads.

## LFI Request

![LFI Request](assets/lfi/lfi-request.png)

The payload attempted to access files outside the intended templates directory.

---

## Confirming LFI

![LFI Response](assets/lfi/etc-passwd-response.png)

Reading `/etc/passwd` confirmed arbitrary file read capability.

### Why This Matters

This demonstrates:

* Directory Traversal.
* Local File Inclusion.
* Improper input validation.

---

## Process Enumeration

![Process Enumeration](assets/lfi/proc-self-cmdline.png)

Reading `/proc/self/cmdline` exposed the application's execution path, allowing discovery of the Flask project directory.

---

# 🧩 Phase 5 — Source Code Analysis

The LFI vulnerability was leveraged to read the Flask application's source code.

## Reading `app.py`

![App Source](assets/source-analysis/app-source-code.png)

### Security Findings

* Flask routes exposed.
* Application logic visible.
* Secrets stored directly in source code.

---

## Flask Route Review

![Routes](assets/source-analysis/flask-routes.png)

Important endpoints identified:

* `fetch_layout`
* Authentication routes.
* Hidden administrative API.

---

## Hardcoded Administrative Secret

![Secret Redacted](assets/source-analysis/hardcoded-secret-redacted.png)

### Finding

A hardcoded administrative token was present inside the application source.

> **Responsible Disclosure**
>
> Secrets and flags have been intentionally redacted in this portfolio.

---

# 🔐 Phase 6 — Administrative Endpoint Discovery

Source code review exposed a hidden API endpoint.

## Hidden Administrative Route

![Admin Endpoint](assets/source-analysis/admin-endpoint.png)

The endpoint required a custom authentication header.

---

## Authenticated Administrative Request

![Admin Request](assets/source-analysis/admin-request.png)

After supplying the recovered administrative token, the endpoint returned the application's SQLite database.

---

# 🗄️ Phase 7 — SQLite Database Analysis

The exported SQLite database was investigated locally.

## Database Download

![Database Download](assets/database/download-database.png)

The administrative endpoint allowed downloading the backend database.

---

## SQLite Investigation

![SQLite Open](assets/database/sqlite-open.png)

### Database Enumeration

* Tables identified.
* Schema inspected.
* User records analyzed.

---

## Database Schema

![Schema](assets/database/users-schema.png)

Important columns included:

* Username.
* Email.
* Phone Number.
* Address.
* Biography.
* Avatar.

---

## Sanitized User Records

![Sanitized Records](assets/database/users-table-redacted.png)

Sensitive data has been intentionally removed from this portfolio.

> **Responsible Disclosure**
>
> Personally identifiable information, administrative secrets, passwords, and challenge flags have been redacted.

---

# 🚨 Security Findings

<table><tr><td>

### Vulnerabilities Identified

| Vulnerability               | Risk     |
| --------------------------- | -------- |
| Local File Inclusion        | High     |
| Directory Traversal         | High     |
| Source Code Disclosure      | Critical |
| Hardcoded Secrets           | Critical |
| Sensitive Data Exposure     | High     |
| Insecure Administrative API | High     |

</td></tr></table>

---

# 🛡️ Phase 8 — Security Impact

## Attack Chain Visualization

![Attack Chain](assets/remediation/attack-flow-diagram.png)

### Security Impact

A single input validation flaw resulted in:

* Source code disclosure.
* Administrative credential exposure.
* Unauthorized database export.
* Sensitive information disclosure.

This illustrates how chained vulnerabilities amplify overall application risk.

---

# 🔧 Phase 9 — Remediation Recommendations

<table><tr><td>

### Secure Coding Improvements

| Issue                          | Recommended Mitigation                                       |
| ------------------------------ | ------------------------------------------------------------ |
| Local File Inclusion           | Validate filenames against an allowlist.                     |
| Directory Traversal            | Canonicalize paths and restrict filesystem access.           |
| Hardcoded Secrets              | Store secrets in environment variables or a secrets manager. |
| Administrative API             | Enforce authentication and role-based authorization.         |
| SQLite Export                  | Restrict database exports and audit administrative actions.  |
| Sensitive Information Exposure | Encrypt sensitive data and implement least privilege access. |

</td></tr></table>

---

# 📚 Skills Demonstrated

This room demonstrates hands-on experience with:

* Linux Enumeration
* Nmap
* Gobuster
* Browser Developer Tools
* HTTP Request Analysis
* Local File Inclusion (LFI)
* Directory Traversal
* Flask Source Code Review
* API Testing
* SQLite Investigation
* Secure Coding Analysis
* Vulnerability Remediation

---

# 📂 Repository Structure

```text
Valenfind-TryHackMe-Walkthrough/
├── README.md
├── Documentation/
│   └── Valenfind_Documentation.md
├── Resources/
│   └── notes.md
├── docs/
│   ├── index.md
│   └── assets/
│       ├── room-banner.png
│       ├── recon/
│       ├── application/
│       ├── lfi/
│       ├── source-analysis/
│       ├── database/
│       └── remediation/
└── LICENSE
```

---

# 📖 Full Technical Documentation

The complete walkthrough, methodology, commands, explanations, and screenshots are available here:

**➡️ [View Complete Documentation](../Documentation/Valenfind_Documentation.md)**

---

# 🏁 Conclusion

The **Valenfind** room demonstrates how a seemingly minor input validation issue can evolve into a complete application compromise when combined with insecure development practices.

This walkthrough showcases a structured penetration testing methodology, emphasizing both **technical exploitation** and **secure software development recommendations** suitable for blue-team and application security learning.

---

<div align="center">

## 👨‍💻 Author

### **Anurag Revankar**

**Cybersecurity | SOC | Web Application Security**

*This repository is maintained as part of my cybersecurity portfolio and documents an educational TryHackMe room completed in a controlled lab environment.*

⭐ **Thank you for visiting my CTF portfolio!**

</div>
