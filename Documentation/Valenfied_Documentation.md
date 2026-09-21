# Valenfind — TryHackMe Walkthrough

## Professional Web Application Security Assessment

**Platform:** TryHackMe

**Room:** Valenfind

**Category:** Web Application Security

**Primary Vulnerability:** Local File Inclusion (LFI)

**Document Version:** 1.0

**Author:** Anurag Revankar

---

## Executive Summary

Valenfind is a web application security challenge hosted on the TryHackMe platform that simulates a vulnerable online dating application developed using the Python Flask framework. The objective of this room is to identify security weaknesses within the application, understand how those weaknesses can be chained together, and safely retrieve the challenge flag inside the authorized lab environment.

This assessment follows a structured penetration testing methodology beginning with reconnaissance, followed by application enumeration, vulnerability discovery, source code analysis, and database inspection. During the assessment, a **Local File Inclusion (LFI)** vulnerability is identified within a dynamic template-loading endpoint. Exploiting this vulnerability exposes internal application files, reveals insecure development practices, and demonstrates how a seemingly minor vulnerability can lead to significant information disclosure.

Rather than focusing only on exploitation, this documentation explains the reasoning behind every step, highlights the associated security risks, and concludes with remediation techniques aligned with OWASP secure development practices.

> **Responsible Disclosure Notice**
>
> This documentation is intended exclusively for educational purposes inside the authorized TryHackMe lab. Challenge flags, API tokens, credentials, and other sensitive values have been intentionally **redacted** throughout this report.

---

# Room Overview

## About the Challenge

Valenfind presents a fictional dating platform where users can create accounts, browse profiles, and interact with other users. Although the application appears functional from a user's perspective, it contains several security flaws that become visible through careful reconnaissance and input analysis.

The room is designed to teach foundational concepts in web application penetration testing, particularly:

* Reconnaissance of web services.
* Endpoint enumeration.
* HTTP request inspection.
* Local File Inclusion (LFI).
* Source code exposure.
* Sensitive information disclosure.
* SQLite database analysis.

---

## Learning Objectives

By completing this room, the following cybersecurity skills are demonstrated:

| Objective               | Description                                                      |
| ----------------------- | ---------------------------------------------------------------- |
| Web Reconnaissance      | Identify exposed services and technologies.                      |
| Directory Enumeration   | Discover hidden application resources.                           |
| HTTP Analysis           | Inspect requests and identify user-controlled parameters.        |
| Vulnerability Discovery | Confirm Local File Inclusion behavior.                           |
| Source Code Review      | Analyze exposed Flask application logic.                         |
| Security Assessment     | Understand the impact of exposed secrets and insecure endpoints. |
| Database Analysis       | Inspect SQLite database contents safely.                         |
| Secure Coding           | Recommend remediation strategies following OWASP guidance.       |

---

# Assessment Methodology

This walkthrough follows a simplified penetration testing lifecycle commonly used during web application security assessments.

## Security Assessment Workflow

```text
Target Discovery
      │
      ▼
Reconnaissance
      │
      ▼
Directory Enumeration
      │
      ▼
Application Exploration
      │
      ▼
HTTP Request Analysis
      │
      ▼
Local File Inclusion Discovery
      │
      ▼
Source Code Review
      │
      ▼
Database Analysis
      │
      ▼
Security Findings & Remediation
```

Each phase builds upon the previous one, demonstrating how attackers gradually expand their understanding of an application before exploiting vulnerabilities.

---

# Lab Environment

## Target Information

| Component        | Details                      |
| ---------------- | ---------------------------- |
| Platform         | TryHackMe                    |
| Application Type | Python Flask Web Application |
| Operating System | Linux                        |
| Web Server       | Flask Development Server     |
| Database         | SQLite                       |
| Authentication   | Session-Based Login          |

## Tools Used During Assessment

| Tool                         | Purpose                                 |
| ---------------------------- | --------------------------------------- |
| Nmap                         | Service and version detection.          |
| Gobuster                     | Directory enumeration.                  |
| Browser Developer Tools      | Inspect network requests and responses. |
| Burp Suite Community Edition | HTTP request modification.              |
| curl                         | Manual API interaction.                 |
| SQLite3                      | Database inspection.                    |
| Linux Terminal               | Enumeration and testing workflow.       |

---

# Attack Surface Overview

Before attempting exploitation, it is important to understand the application's exposed functionality.

The application includes several features available to authenticated users:

* User registration.
* Login authentication.
* Dashboard containing public user profiles.
* Individual profile pages.
* Profile customization.
* Dynamic profile theme selection.
* API endpoints used by the frontend.

The dynamic theme-loading feature eventually becomes the primary attack surface investigated during this room.

---

# Phase 1 — Initial Reconnaissance

## Objective

The first step in every penetration test is identifying the services running on the target system.

Rather than immediately searching for vulnerabilities, reconnaissance answers a fundamental question:

> **What services are exposed, and what technologies are running?**

---

## Service Enumeration with Nmap

The assessment begins with a standard Nmap scan to identify open ports and service versions.

```bash
nmap -sC -sV <TARGET-IP>
```

### Command Breakdown

| Option | Purpose                                               |
| ------ | ----------------------------------------------------- |
| `-sC`  | Executes default NSE scripts for basic enumeration.   |
| `-sV`  | Detects service versions running on discovered ports. |

---

## Screenshot 1.1 — Initial Nmap Scan

> **Screenshot Location**

```text
docs/assets/recon/nmap-scan.png
```

Add the screenshot from your repository here.

---

## Findings

The scan identifies a web service listening on **Port 5000**.

### Security Observation

Port **5000** is frequently associated with Python Flask development servers. While this alone is not a vulnerability, identifying the underlying technology helps guide later enumeration techniques.

### Why This Matters

Knowing the application uses Flask suggests several possibilities:

* Template rendering.
* Static resource directories.
* API routes.
* Session-based authentication.
* Python application files stored locally.

This information becomes valuable during later exploitation stages.

---

## Initial Attack Surface Summary

| Observation          | Security Value                      |
| -------------------- | ----------------------------------- |
| Port 5000 Open       | Web application exposed.            |
| HTTP Service Running | Accessible attack surface.          |
| Flask Fingerprint    | Indicates Python-based application. |

---

## Security Note

Reconnaissance should always be **non-destructive**. At this stage, no exploitation is performed; only publicly exposed information is collected to understand the target environment.

---

# Phase 2 — Directory Enumeration

## Objective

Once a web application is identified, the next goal is discovering hidden directories, resources, and endpoints that are not immediately visible through the user interface.

Directory enumeration often reveals:

* Administrative panels.
* API endpoints.
* Static assets.
* Hidden pages.
* Backup files.

---

## Directory Discovery with Gobuster

Gobuster performs a dictionary-based scan against the web server.

```bash
gobuster dir \
-u http://<TARGET-IP>:5000/ \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
-t 80
```

### Command Breakdown

| Option  | Purpose                                         |
| ------- | ----------------------------------------------- |
| `dir`   | Directory enumeration mode.                     |
| `-u`    | Target URL.                                     |
| `-w`    | Wordlist containing directory names.            |
| `-t 80` | Uses 80 concurrent threads for faster scanning. |

---

## Screenshot 2.1 — Gobuster Enumeration

> **Screenshot Location**

```text
docs/assets/recon/gobuster.png
```

Insert Gobuster results here.

---

## Enumeration Findings

Directory enumeration confirms that the application exposes several accessible resources used by the frontend.

Although not every discovered endpoint is vulnerable, enumeration provides a roadmap for manual exploration.

### Security Observation

Enumeration reduces the application's "unknown attack surface." Hidden functionality often exists behind routes that are never linked through the homepage.

---

## Manual Application Access

After enumeration, the application is opened inside the browser.

```
http://<TARGET-IP>:5000/
```

The landing page displays the **Valenfind** dating application interface.

---

## Screenshot 2.2 — Valenfind Homepage

> **Screenshot Location**

```text
docs/assets/application/homepage.png
```

Insert the homepage screenshot here.

---

## Initial Interface Review

The homepage reveals a modern dating application with authentication features and user interaction capabilities.

Visible functionality includes:

* Registration page.
* Login page.
* User dashboard.
* Profile pages.
* Theme customization.

At this point, nothing appears obviously vulnerable, reinforcing the importance of careful exploration instead of making assumptions.

---

## Reconnaissance Summary

| Activity              | Result                                       |
| --------------------- | -------------------------------------------- |
| Service Scan          | Flask web application discovered.            |
| Version Detection     | HTTP service identified on Port 5000.        |
| Directory Enumeration | Multiple application resources discovered.   |
| Homepage Analysis     | User-facing application successfully mapped. |

---

## Key Takeaways from Reconnaissance

* Reconnaissance establishes the technology stack before exploitation.
* Enumeration helps identify hidden application functionality.
* Flask applications frequently expose API routes that deserve additional inspection.
* No exploitation has occurred yet; the assessment remains in the information gathering phase.

---

## Phase 3 — Application Exploration

### Objective

After identifying the exposed web application through reconnaissance, the next step is to understand how the application behaves from a legitimate user's perspective. Exploring application functionality before attempting exploitation is a critical penetration testing practice because many vulnerabilities only become visible after authentication or through normal user workflows.

The goal during this phase is to map the application's features, identify user-controlled inputs, observe network communication, and locate functionality that interacts with backend resources.

---

## Understanding the Application

Valenfind is presented as a social dating application where users can:

* Register a new account.
* Authenticate using username and password.
* Complete their personal profile.
* Browse public user profiles.
* Like other users.
* Customize the appearance of profile pages.

At first glance, the application behaves like a standard Flask web application with session-based authentication.

---

## Step 3.1 — Registering a New Account

The first interaction with the application is creating a new user account.

Creating an account provides authenticated access to additional application functionality that is unavailable from the landing page.

### Screenshot 3.1 — Registration Page

**Screenshot Location**

```text id="xmhjlc"
docs/assets/application/register.png
```

*Insert your registration page screenshot here.*

### Observation

The registration page accepts several user-controlled inputs including:

* Username
* Password

After successful registration, the application redirects the user to complete additional profile information.

### Security Observation

Registration functionality often introduces opportunities for testing:

* Input validation.
* Username uniqueness.
* Authentication workflow.
* Session handling.

At this stage, no security issues were observed during account creation.

---

## Step 3.2 — Completing the User Profile

After registration, the application prompts users to complete their profile before accessing the dashboard.

### Screenshot 3.2 — Complete Profile Page

**Screenshot Location**

```text id="ofdwf0"
docs/assets/application/complete-profile.png
```

### Information Collected

The profile form contains several editable fields:

| Field        | Purpose                     |
| ------------ | --------------------------- |
| Real Name    | Display information.        |
| Email        | User contact information.   |
| Phone Number | Personal information.       |
| Address      | User profile data.          |
| Biography    | Public profile description. |

### Why Explore This?

Editable forms are valuable during security assessments because they frequently interact with backend databases and rendering engines.

Potential testing targets include:

* Stored input.
* Rendering behavior.
* Validation.
* Backend requests.

---

## Step 3.3 — Logging Into the Application

After profile completion, authentication is performed using the newly created credentials.

### Screenshot 3.3 — Login Page

**Screenshot Location**

```text id="zlgw9m"
docs/assets/application/login.png
```

Successful authentication redirects the user to the dashboard.

### Authentication Behavior

The application uses session-based authentication.

Successful login creates an authenticated session that allows access to additional pages and API functionality.

### Security Observation

Authenticated areas frequently expose:

* Hidden endpoints.
* Administrative features.
* API requests.
* Additional attack surface.

This is why authenticated testing is an important part of web application assessments.

---

## Step 3.4 — Dashboard Exploration

The dashboard displays multiple user profiles available within the application.

### Screenshot 3.4 — User Dashboard

**Screenshot Location**

```text id="yxprw8"
docs/assets/application/dashboard.png
```

### Features Identified

The dashboard allows users to:

* Browse profiles.
* View biographies.
* Like other users.
* Open individual profile pages.

### Initial Observations

Each profile contains dynamic content including:

* Avatar image.
* Username.
* Biography.
* Theme selection.
* Interaction buttons.

Nothing immediately appears vulnerable through the user interface alone.

---

## Step 3.5 — Exploring User Profiles

Selecting an individual profile loads a dedicated profile page.

### Screenshot 3.5 — Profile Page

**Screenshot Location**

```text id="24jaov"
docs/assets/application/profile-page.png
```

### New Functionality Observed

A dropdown menu labeled **Profile Theme** allows users to change how profile information is displayed.

Available themes include:

* Classic Romance
* Modern Dark
* Cupid's Choice

Changing the theme dynamically updates the profile content without reloading the page.

### Security Observation

Dynamic content loading often communicates with backend API endpoints, making it an excellent candidate for further investigation.

---

## Phase Summary

At the end of application exploration, we have identified:

* Authentication workflow.
* User-controlled profile functionality.
* Dynamic theme rendering.
* Client-side requests triggered by UI interactions.

These observations guide the next phase: network traffic analysis.

---

# Phase 4 — HTTP Request Analysis

## Objective

Rather than guessing vulnerabilities, we inspect the application's HTTP requests to understand how the frontend communicates with the backend.

Modern web applications frequently expose REST API endpoints that process user-controlled parameters.

---

## Inspecting Network Requests

Browser Developer Tools provide visibility into every request generated by the application.

### Tools Used

* Firefox Developer Tools
* Chrome Developer Tools

### Steps Performed

1. Open Developer Tools (`F12`).
2. Navigate to the **Network** tab.
3. Refresh the profile page.
4. Change the profile theme.
5. Observe newly generated requests.

### Screenshot 4.1 — Network Traffic

**Screenshot Location**

```text id="pv8cpb"
docs/assets/application/network-inspector.png
```

---

## Interesting API Endpoint Discovered

While switching themes, the browser sends a request similar to:

```http id="ijhsy8"
GET /api/fetch_layout?layout=theme_classic.html
```

### Parameter Breakdown

| Parameter | Purpose                                       |
| --------- | --------------------------------------------- |
| `layout`  | Specifies which HTML layout should be loaded. |

The application fetches HTML content from the backend and renders it inside the user's profile.

### Why Is This Interesting?

Whenever user input determines which server-side file is loaded, penetration testers immediately ask:

> **Can user input influence filesystem access?**

This becomes the primary hypothesis for the next phase.

---

## Understanding Dynamic Template Loading

The profile page does not contain every theme inside the HTML source.

Instead:

1. User selects a theme.
2. Browser sends the selected theme name.
3. Backend returns HTML.
4. Frontend injects the HTML into the page.

### Simplified Workflow

```text id="vynbqd"
User Selects Theme
        │
        ▼
Browser Sends Request
        │
        ▼
GET /api/fetch_layout
        │
        ▼
Server Reads Template File
        │
        ▼
HTML Returned to Browser
        │
        ▼
Profile Updated Dynamically
```

### Security Observation

Applications that read files based on user input require strict validation.

Without proper validation, attackers may request files outside the intended templates directory.

---

## Why This Endpoint Deserves Testing

Indicators suggesting possible file inclusion include:

* File name passed directly as input.
* HTML returned as response.
* Dynamic rendering.
* No visible client-side validation.

These characteristics are commonly associated with Local File Inclusion vulnerabilities.

---

## Phase Summary

| Finding                         | Security Relevance                             |
| ------------------------------- | ---------------------------------------------- |
| Dynamic API endpoint discovered | Backend file-loading functionality identified. |
| User-controlled parameter found | Potential attack surface.                      |
| HTML loaded from server         | Possible filesystem interaction.               |

The next step is validating whether the parameter can access unintended files.

---

# Phase 5 — Local File Inclusion Discovery

## Objective

Determine whether the `layout` parameter can be manipulated to access arbitrary files stored on the server.

Local File Inclusion (LFI) occurs when an application reads files from the filesystem using unsanitized user input.

---

## Understanding Local File Inclusion

### What is LFI?

Local File Inclusion allows attackers to access files stored locally on the web server.

Instead of loading intended templates, attackers attempt to traverse directories and read sensitive operating system or application files.

### Common Indicators

* File path parameters.
* Template inclusion.
* Directory traversal sequences.
* Unexpected file contents returned.

---

## Testing Directory Traversal

A classic LFI validation payload targets a universally readable Linux file.

### Test Payload

```text id="ttspmq"
../../../../etc/passwd
```

The modified request becomes:

```http id="6of39y"
GET /api/fetch_layout?layout=../../../../etc/passwd
```

### Why `/etc/passwd`?

The `/etc/passwd` file exists on nearly every Linux system and contains user account information.

Security testers use this file because:

* It confirms filesystem access.
* It is readable by many processes.
* It does not modify the target.

---

## Screenshot 5.1 — LFI Payload

**Screenshot Location**

```text id="j9z3p0"
docs/assets/lfi/lfi-request.png
```

---

## LFI Validation Result

The application responds with the contents of `/etc/passwd`.

### Screenshot 5.2 — `/etc/passwd` Response

**Screenshot Location**

```text id="e1eqqw"
docs/assets/lfi/etc-passwd.png
```

### Security Finding

The response confirms that user-controlled input is being passed directly into backend file operations without sufficient sanitization.

### Vulnerability Confirmed

**Local File Inclusion (LFI)**

---

## Why This Vulnerability Is Critical

Successful LFI can expose:

| Resource                | Risk                               |
| ----------------------- | ---------------------------------- |
| Operating system files  | Information disclosure.            |
| Application source code | Secret exposure.                   |
| Configuration files     | Credential leakage.                |
| Logs                    | Sensitive operational information. |
| Environment files       | API keys and secrets.              |

LFI often becomes the first step in a longer attack chain.

---

## Root Cause Analysis

The application trusts user input when constructing file paths.

A secure implementation should:

* Restrict filenames.
* Validate extensions.
* Normalize paths.
* Reject traversal sequences (`../`).

Without these protections, attackers can escape the intended templates directory.

---

## Security Observation

> **Directory traversal is not the vulnerability itself.**

The vulnerability is the application's failure to validate filesystem paths before opening files requested by the client.

---

## Expanding the Attack Surface

After confirming LFI, the next objective becomes identifying valuable internal files.

Common targets include:

| File                    | Purpose                      |
| ----------------------- | ---------------------------- |
| `/proc/self/cmdline`    | Running process information. |
| Application source code | Backend logic.               |
| Configuration files     | Secrets and credentials.     |
| Environment files       | Tokens and API keys.         |

Instead of guessing randomly, attackers prioritize files that reveal application behavior.

---

## Testing Process Information

A useful reconnaissance file inside Linux is:

```text id="q4w24d"
../../../../proc/self/cmdline
```

This file reveals how the running application was launched and helps identify the application's installation path.

### Screenshot 5.3 — Process Information

**Screenshot Location**

```text id="x9ctqk"
docs/assets/lfi/proc-self-cmdline.png
```

### Observation

The returned output provides information about the running Flask process and assists in locating the application's source directory.

### Why This Matters

Understanding the application path allows us to target source code files instead of guessing filenames.

---

## Phase 5 Summary

| Activity                      | Result                                     |
| ----------------------------- | ------------------------------------------ |
| Parameter analysis            | User-controlled file parameter identified. |
| Directory traversal test      | Successful.                                |
| `/etc/passwd` accessed        | LFI confirmed.                             |
| Process information retrieved | Application path identified.               |
| Next Target                   | Flask application source code.             |

---

## Security Assessment Notes

### Vulnerability Severity

**High**

### Impact

* Arbitrary file disclosure.
* Internal application exposure.
* Potential credential leakage.
* Increased attack surface through source code access.

### OWASP Classification

* Security Misconfiguration.
* Broken Access Control.
* Sensitive Information Exposure.

---

# Phase 6 — Source Code Analysis

## Objective

After confirming the Local File Inclusion vulnerability, the next objective is identifying valuable files stored on the server. Rather than reading random files, penetration testers prioritize application source code because it often reveals authentication mechanisms, hidden endpoints, configuration mistakes, and sensitive secrets.

The goal of this phase is to safely analyze the Flask application's source code and understand how the vulnerability expands into a larger attack chain.

---

## Why Source Code Matters

Source code provides visibility into how an application works internally. Unlike the user interface, source code exposes:

* Application routes.
* Authentication logic.
* API endpoints.
* Database connections.
* File handling implementation.
* Hardcoded configuration values.

When attackers obtain backend code through LFI, they gain insight that is normally unavailable from the browser.

---

## Identifying the Application Path

During the previous phase, reading `/proc/self/cmdline` revealed information about the running Flask process and helped identify the application's installation directory.

This significantly narrows the search for important files.

### Screenshot 6.1 — Application Path Discovery

**Screenshot Location**

```text id="afl9gs"
docs/assets/lfi/proc-self-cmdline.png
```

---

## Accessing the Flask Source Code

Using the discovered application path, the next target becomes the primary Flask application file.

### Target File

```text id="m1pcme"
../../../../opt/Valenfind/app.py
```

### Screenshot 6.2 — Reading app.py

**Screenshot Location**

```text id="eqm9jg"
docs/assets/source-analysis/app-source.png
```

The response contains the application's backend source code.

---

## Initial Source Code Review

Rather than reading the entire file immediately, the assessment focuses on identifying security-relevant sections.

### Areas Reviewed

| Component              | Why It Matters                     |
| ---------------------- | ---------------------------------- |
| Flask Routes           | Discover hidden functionality.     |
| Database Configuration | Locate sensitive data.             |
| Authentication Logic   | Understand access control.         |
| File Handling          | Identify LFI implementation.       |
| API Endpoints          | Find administrative functionality. |

This approach mirrors how source code reviews are typically performed during web application assessments.

---

## Application Architecture Overview

The retrieved source code reveals a relatively simple Flask application.

### Core Components Identified

* Flask web server.
* Session-based authentication.
* SQLite database backend.
* User profile routes.
* Theme-loading API endpoint.
* Administrative export functionality.

### Simplified Architecture

```text id="fgd6o5"
Browser
   │
   ▼
Flask Application
   │
   ├── Authentication Routes
   ├── Profile Routes
   ├── Theme Loader API
   └── Admin API
         │
         ▼
      SQLite Database
```

This architecture becomes important for understanding how one vulnerability affects multiple application components.

---

## Reviewing Route Definitions

The Flask application defines multiple routes responsible for user functionality.

Examples include:

* Homepage.
* Registration.
* Login.
* Dashboard.
* Profile management.
* Theme loading.
* Administrative API.

### Screenshot 6.3 — Flask Routes

**Screenshot Location**

```text id="s9yk0r"
docs/assets/source-analysis/routes.png
```

### Security Observation

Route discovery often uncovers functionality that is inaccessible through normal navigation.

Hidden routes deserve additional inspection because they frequently expose administrative operations.

---

## Understanding the Theme Loading Function

One of the most important functions inside the application handles profile theme loading.

### Simplified Logic

The endpoint accepts a filename supplied by the client and attempts to read that file from disk before returning its contents.

### Security Observation

The application attempts to construct a filesystem path using user-controlled input.

Without strict validation, directory traversal sequences can escape the intended directory.

### Root Cause

* User input influences filesystem operations.
* No canonical path validation.
* No allowlist enforcement.

This is the implementation mistake responsible for the LFI vulnerability confirmed earlier.

---

## Security Weakness Identified

### Local File Inclusion Root Cause

The application trusts the `layout` parameter and attempts to read files relative to a templates directory.

A secure implementation should instead:

* Allow only predefined template names.
* Reject traversal characters.
* Resolve canonical paths.
* Verify the requested file remains inside the templates directory.

---

## Reviewing Authentication Logic

The login implementation is also reviewed during source analysis.

### Findings

Authentication uses:

* Username lookup.
* Password comparison.
* Session creation after successful login.

### Security Observation

Although authentication works for legitimate users, source code visibility exposes implementation details that attackers should never see.

---

## Sensitive Information Discovery

One section of the application contains a hardcoded administrative secret.

### Screenshot 6.4 — Hardcoded Secret (Redacted)

**Screenshot Location**

```text id="hygv8x"
docs/assets/source-analysis/admin-secret.png
```

### Portfolio Version

```python id="2wpk7e"
ADMIN_API_KEY = "<REDACTED_ADMIN_TOKEN>"
```

> **Important:** The real challenge value has been intentionally removed from this repository.

---

## Why Hardcoded Secrets Are Dangerous

Hardcoded credentials create significant security risks.

### Risks Include

| Risk                   | Description                                    |
| ---------------------- | ---------------------------------------------- |
| Credential Exposure    | Secrets become visible if source code leaks.   |
| Privilege Escalation   | Attackers gain administrative capabilities.    |
| Source Control Leakage | Secrets may be committed to repositories.      |
| Secret Reuse           | Credentials may be reused across environments. |

### OWASP Recommendation

Secrets should be stored using:

* Environment variables.
* Secret management services.
* Configuration outside source control.

---

## Additional Source Code Findings

The application source reveals references to:

* SQLite database file.
* Administrative routes.
* Template rendering.
* File download functionality.

Each of these becomes a potential target for further investigation.

---

## Security Observation

Source code exposure dramatically increases attacker knowledge.

Even if attackers cannot immediately exploit another vulnerability, source code often reveals:

* Hidden endpoints.
* API keys.
* Internal filenames.
* Business logic.
* Authorization mistakes.

This is why protecting source code is considered a critical security practice.

---

# Phase 7 — Administrative Endpoint Discovery

## Objective

After identifying a hardcoded administrative secret, the next task is understanding where it is used within the application.

Rather than guessing endpoints, source code analysis provides direct visibility into administrative functionality.

---

## Discovering Hidden Routes

Reviewing Flask routes reveals an endpoint that is not linked anywhere in the application's user interface.

### Screenshot 7.1 — Hidden Administrative Endpoint

**Screenshot Location**

```text id="fpz7r9"
docs/assets/source-analysis/admin-endpoint.png
```

### Simplified Endpoint

```text id="s53l4u"
/api/admin/export_db
```

### Why This Is Important

This endpoint is inaccessible through normal navigation but still exists inside the application.

Hidden functionality is not secure simply because it is undocumented.

---

## Authentication Mechanism

The administrative endpoint validates a custom HTTP request header before granting access.

### Simplified Logic

```text id="kvwtqr"
Client Request
      │
      ▼
Check HTTP Header
      │
      ▼
Compare With Admin Secret
      │
      ▼
Allow or Deny Access
```

### Security Observation

Security depends entirely on possession of a secret stored inside the application's source code.

Once source code is exposed through LFI, this protection becomes ineffective.

---

## Why This Becomes Vulnerability Chaining

This room demonstrates an important security concept:

### Vulnerability Chain

```text id="jb8gth"
Local File Inclusion
        │
        ▼
Read Source Code
        │
        ▼
Discover Hidden Endpoint
        │
        ▼
Discover Hardcoded Secret
        │
        ▼
Access Administrative Functionality
```

Rather than a single vulnerability causing compromise, multiple weaknesses combine into a larger attack path.

---

## Security Assessment

### Weaknesses Combined

| Weakness                       | Impact                                 |
| ------------------------------ | -------------------------------------- |
| Local File Inclusion           | Source code disclosure.                |
| Hardcoded Secret               | Administrative credential exposure.    |
| Hidden Administrative Endpoint | Unauthorized privileged functionality. |

This demonstrates how defense-in-depth fails when multiple insecure practices exist simultaneously.

---

## Accessing the Administrative Functionality

Once the endpoint and authentication method are understood, the assessment interacts with the endpoint using standard HTTP tools.

Possible tools include:

* Browser extensions.
* Burp Suite.
* curl.
* Postman.

### Screenshot 7.2 — Administrative Request

**Screenshot Location**

```text id="zk2b1r"
docs/assets/source-analysis/admin-request.png
```

---

## Security Observation

The request includes a custom authentication header.

For portfolio purposes, the sensitive value is redacted.

### Portfolio Example

```http id="ajppvl"
GET /api/admin/export_db HTTP/1.1
Host: <TARGET-IP>:5000

X-Valentine-Token: <REDACTED_ADMIN_TOKEN>
```

No real challenge secrets are published.

---

## Administrative Endpoint Response

When valid authentication is supplied, the endpoint returns a downloadable SQLite database.

### Screenshot 7.3 — Database Download

**Screenshot Location**

```text id="j0ylg1"
docs/assets/database/download-db.png
```

### Observation

The server responds with a downloadable database file instead of HTML.

This indicates the endpoint exposes backend application data directly.

---

## Security Impact

### Sensitive Data Exposure

The exported database may contain:

* User accounts.
* Email addresses.
* Phone numbers.
* Profile information.
* Administrative records.

### Why This Is Critical

Applications should never expose production databases through downloadable endpoints without strong authorization controls.

---

## Security Finding Summary

### Finding 1 — Hidden Administrative Functionality

**Severity:** High

**Description**

An undocumented administrative endpoint exists within the application.

**Risk**

Unauthorized users gaining administrative credentials can access privileged functionality.

---

### Finding 2 — Hardcoded Administrative Secret

**Severity:** High

**Description**

Administrative authentication depends on a secret embedded directly inside application source code.

**Risk**

Source code disclosure immediately exposes privileged authentication material.

---

### Finding 3 — Sensitive Database Export

**Severity:** High

**Description**

Administrative functionality allows downloading the application's SQLite database.

**Risk**

Sensitive user information becomes accessible after administrative authentication is bypassed.

---

## Blue Team Perspective

Security teams should monitor:

* Requests containing unusual authentication headers.
* Unexpected access to administrative routes.
* Database download activity.
* Requests originating from authenticated but low-privilege users.

Proper logging would help detect exploitation attempts early.

---

## Phase Summary

| Activity                         | Result                                            |
| -------------------------------- | ------------------------------------------------- |
| Source code retrieved            | Flask application analyzed.                       |
| Hardcoded secret identified      | Sensitive authentication material exposed.        |
| Hidden endpoint discovered       | Administrative functionality mapped.              |
| Administrative endpoint analyzed | Database export behavior identified.              |
| Next Phase                       | SQLite database inspection and security findings. |

---

## Lessons Learned

* Source code exposure often reveals additional attack paths.
* Hidden endpoints should still implement robust authorization.
* Hardcoded secrets undermine authentication security.
* Multiple low-level weaknesses can combine into a high-impact vulnerability chain.

---


# Phase 8 — SQLite Database Analysis

## Objective

After identifying the hidden administrative endpoint, the next phase is understanding the information exposed by the application's SQLite database. Rather than treating the database as the final objective, this section explains why unrestricted database access represents a critical security risk.

The assessment focuses on reviewing database structure, understanding exposed information, and identifying the overall security impact without revealing protected challenge content.

---

## Understanding SQLite in Flask Applications

SQLite is a lightweight relational database commonly used by Flask applications during development and small deployments.

### Why Developers Use SQLite

* Easy setup.
* No separate database server required.
* File-based storage.
* Suitable for prototypes and small applications.

### Security Consideration

Because SQLite stores the entire database inside a single file, exposing that file can reveal all stored application data.

---

## Database Download

The administrative endpoint responds with a downloadable SQLite database file.

### Screenshot 8.1 — Database Download Response

**Screenshot Location**

```text
docs/assets/database/download-db.png
```

*Insert your screenshot showing the database download response.*

### Observation

The application returns a downloadable database instead of rendering HTML content. This indicates the endpoint exposes backend storage directly to authenticated requests.

### Security Observation

Administrative endpoints that expose raw databases should implement:

* Strong authentication.
* Authorization checks.
* Audit logging.
* Least privilege access.

---

## Opening the Database

The downloaded SQLite database can be inspected using the SQLite command-line client.

### Command Used

```bash
sqlite3 cupid.db
```

### Basic Enumeration Commands

```sql
.tables
.schema users
SELECT * FROM users;
```

### Screenshot 8.2 — SQLite Database Inspection

**Screenshot Location**

```text
docs/assets/database/sqlite-open.png
```

---

## Database Structure

The database contains a table responsible for storing user information.

### Table Identified

| Table   | Purpose                          |
| ------- | -------------------------------- |
| `users` | Stores application user records. |

### Screenshot 8.3 — Users Table Schema

**Screenshot Location**

```text
docs/assets/database/users-schema.png
```

### Security Observation

Schema review helps identify:

* Personally identifiable information (PII).
* Authentication data.
* Administrative accounts.
* Sensitive metadata.

---

## Reviewing User Records

The `users` table stores multiple user profiles used by the application.

### Portfolio Version (Sanitized)

| Username        | Email        | Bio                          | Status              |
| --------------- | ------------ | ---------------------------- | ------------------- |
| romeo_montague  | `<redacted>` | Looking for my Juliet...     | Standard User       |
| cleopatra_queen | `<redacted>` | Public profile description   | Standard User       |
| sherlock_h      | `<redacted>` | Public profile description   | Standard User       |
| cupid           | `<redacted>` | **Sensitive value redacted** | Administrative User |

> Challenge-specific values have been intentionally removed.

### Screenshot 8.4 — Sanitized Database Records

**Screenshot Location**

```text
docs/assets/database/users-table.png
```

---

## Security Impact Assessment

The database contains multiple categories of sensitive information.

### Exposed Information Categories

| Data Type              | Security Impact                      |
| ---------------------- | ------------------------------------ |
| Usernames              | Account enumeration.                 |
| Email Addresses        | Personally identifiable information. |
| Phone Numbers          | Privacy exposure.                    |
| Addresses              | Sensitive user information.          |
| User Bios              | Public profile metadata.             |
| Administrative Records | Increased attack surface.            |

### Risk Assessment

Unauthorized database disclosure can lead to:

* Privacy violations.
* Credential exposure.
* User enumeration.
* Administrative account discovery.
* Additional attack chaining opportunities.

---

## Why Database Exposure Is Critical

Database exposure is significantly more impactful than reading a single configuration file.

### Impact Comparison

| Vulnerability        | Typical Impact                              |
| -------------------- | ------------------------------------------- |
| LFI                  | Read individual files.                      |
| Source Code Exposure | Discover secrets and application logic.     |
| Database Exposure    | Access sensitive application data at scale. |

This room demonstrates how vulnerabilities can escalate from information disclosure into broader application compromise.

---

## Security Observation

The administrative endpoint effectively bypasses application-layer access controls by returning the backend database directly.

A production application should never expose complete database files through downloadable endpoints.

---

# Phase 9 — Security Findings

## Finding 1 — Local File Inclusion (LFI)

### Severity

**High**

### Description

A user-controlled parameter allowed the application to read arbitrary files from the underlying operating system through directory traversal.

### Evidence

* Dynamic template loading.
* Traversal payload accepted.
* Server returned unintended filesystem content.

### Impact

* Arbitrary file disclosure.
* Source code exposure.
* Sensitive configuration disclosure.

### Root Cause

Insufficient validation of filesystem paths supplied by user input.

---

## Finding 2 — Directory Traversal

### Severity

**High**

### Description

The application failed to normalize or validate requested paths before reading files.

### Impact

Attackers escaped the intended templates directory and accessed operating system files.

### Recommended Fix

* Normalize filesystem paths.
* Reject traversal sequences.
* Verify canonical path remains inside approved directory.

---

## Finding 3 — Hardcoded Administrative Secret

### Severity

**High**

### Description

Administrative authentication relied on a secret embedded directly inside the application source code.

### Impact

Source code disclosure immediately exposed privileged authentication material.

### Recommended Fix

Store secrets using:

* Environment variables.
* Secret management services.
* External configuration files excluded from source control.

---

## Finding 4 — Hidden Administrative Endpoint

### Severity

**Medium–High**

### Description

Administrative functionality existed outside normal application navigation.

### Risk

Hidden functionality is still discoverable through source code analysis or enumeration.

### Recommended Fix

* Enforce authentication.
* Enforce authorization.
* Restrict administrative routes by role.

---

## Finding 5 — Sensitive Database Export

### Severity

**High**

### Description

A privileged endpoint returned the complete SQLite database.

### Impact

Exposure of application user data and administrative records.

### Recommended Fix

* Remove direct database download functionality.
* Export sanitized reports instead of raw databases.
* Require strong authorization and auditing.

---

# Vulnerability Chain Summary

The exploitation path demonstrates how multiple weaknesses combine into a single attack chain.

```text
User Input
     │
     ▼
Local File Inclusion
     │
     ▼
Read Flask Source Code
     │
     ▼
Discover Administrative Secret
     │
     ▼
Discover Hidden Endpoint
     │
     ▼
Download SQLite Database
     │
     ▼
Sensitive Information Disclosure
```

### Security Lesson

No single vulnerability completely compromised the application.

Instead, insecure development practices combined to create a much larger security impact.

---

# OWASP Top 10 Mapping

This room maps closely to several OWASP Top 10 categories.

| OWASP Category                               | Application Example                             |
| -------------------------------------------- | ----------------------------------------------- |
| **Broken Access Control**                    | Administrative endpoint exposure.               |
| **Cryptographic Failures**                   | Hardcoded secret management.                    |
| **Injection / Path Manipulation**            | Directory traversal through template parameter. |
| **Security Misconfiguration**                | Unsafe filesystem access.                       |
| **Identification & Authentication Failures** | Weak administrative authentication design.      |

### Why This Mapping Matters

OWASP categories help developers prioritize remediation based on common web application security risks.

---

# Phase 10 — Remediation Recommendations

## 1. Prevent Local File Inclusion

### Current Risk

User input directly determines which file is read from disk.

### Recommended Fix

Use an allowlist of permitted templates.

**Example Concept**

```python
allowed_templates = {
    "classic": "theme_classic.html",
    "modern": "theme_modern.html",
    "romance": "theme_romance.html"
}
```

Never allow arbitrary filenames supplied by users.

---

## 2. Validate Filesystem Paths

### Recommended Controls

* Normalize paths.
* Reject `../`.
* Restrict access to templates directory only.
* Verify canonical paths before opening files.

---

## 3. Remove Hardcoded Secrets

### Replace With

* Environment variables.
* Configuration management.
* Secret vaults.
* Deployment-specific configuration.

### Benefits

* Secrets remain outside source code.
* Safer deployments.
* Easier credential rotation.

---

## 4. Secure Administrative Endpoints

### Recommendations

* Role-Based Access Control (RBAC).
* Server-side authorization checks.
* Multi-factor authentication for administrative operations.
* Audit logging.

---

## 5. Protect Database Files

### Recommendations

* Store database outside web-accessible paths.
* Disable direct download functionality.
* Encrypt sensitive backups.
* Restrict filesystem permissions.

---

## 6. Improve Flask Security Configuration

### Recommended Settings

| Configuration  | Recommendation                           |
| -------------- | ---------------------------------------- |
| Debug Mode     | Disabled in production.                  |
| Secret Key     | Loaded from environment variables.       |
| Error Messages | Avoid revealing filesystem paths.        |
| Logging        | Record administrative activity securely. |

---

# Secure Development Best Practices

## Input Validation

Always validate:

* Query parameters.
* Form inputs.
* Uploaded filenames.
* Template names.

---

## Principle of Least Privilege

Administrative functionality should only be available to authorized users with the minimum required permissions.

---

## Sensitive Information Handling

Avoid exposing:

* API keys.
* Tokens.
* Passwords.
* Configuration files.
* Database files.

---

## Error Handling

Replace verbose backend errors with generic responses to avoid leaking filesystem information.

---

# Lessons Learned

## Technical Lessons

This room demonstrates several foundational cybersecurity concepts.

### Key Takeaways

* Reconnaissance provides valuable context before exploitation.
* Authenticated functionality often exposes additional attack surface.
* Dynamic file-loading mechanisms require strict validation.
* Local File Inclusion can expose much more than operating system files.
* Source code disclosure frequently reveals hidden endpoints and secrets.
* Hardcoded secrets dramatically increase the impact of source code exposure.
* Database exposure represents a serious confidentiality risk.

---

## Blue Team Perspective

Security teams should monitor for:

* Requests containing traversal sequences.
* Repeated failed template requests.
* Access to undocumented administrative routes.
* Database download activity.
* Abnormal authenticated requests.

Logging these events improves detection and incident response.

---

## Red Team Perspective

Skills practiced during this room include:

* Web reconnaissance.
* Directory enumeration.
* HTTP request inspection.
* Local File Inclusion testing.
* Source code analysis.
* SQLite database inspection.
* Vulnerability chaining.

These skills are directly applicable to beginner web penetration testing labs.

---

# Key Commands Used During Assessment

## Reconnaissance

```bash
nmap -sC -sV <TARGET-IP>
```

## Directory Enumeration

```bash
gobuster dir \
-u http://<TARGET-IP>:5000 \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

## SQLite Inspection

```bash
sqlite3 cupid.db

.tables
.schema users
SELECT * FROM users;
.quit
```

> Sensitive commands containing challenge-specific authentication material have been intentionally omitted.

---

# Screenshots Used Throughout This Documentation

| Phase                   | Screenshot Folder              |
| ----------------------- | ------------------------------ |
| Reconnaissance          | `docs/assets/recon/`           |
| Enumeration             | `docs/assets/recon/`           |
| Application Exploration | `docs/assets/application/`     |
| HTTP Request Analysis   | `docs/assets/application/`     |
| Local File Inclusion    | `docs/assets/lfi/`             |
| Source Code Analysis    | `docs/assets/source-analysis/` |
| Administrative Endpoint | `docs/assets/source-analysis/` |
| Database Analysis       | `docs/assets/database/`        |
| Remediation             | `docs/assets/remediation/`     |

This folder structure keeps GitHub Pages documentation organized and easy to maintain.

---

# Conclusion

The **Valenfind** TryHackMe room provides an excellent introduction to web application penetration testing by demonstrating how small security weaknesses can combine into a meaningful attack chain.

Beginning with reconnaissance and endpoint enumeration, the assessment gradually identifies a vulnerable file-loading endpoint, confirms a **Local File Inclusion (LFI)** vulnerability, analyzes exposed Flask source code, discovers insecure administrative functionality, and demonstrates the security impact of exposing backend application data.

Beyond exploitation, this walkthrough emphasizes secure development principles and OWASP-aligned remediation strategies. The room reinforces an important lesson in application security: protecting individual endpoints is not enough if secrets, filesystem access, and authorization mechanisms are implemented insecurely.

---

# Responsible Disclosure & Portfolio Notice

This documentation was created for educational purposes as part of a cybersecurity portfolio documenting practical labs completed on **TryHackMe**.

The following information has been intentionally **redacted** throughout this repository:

* TryHackMe challenge flag.
* Administrative API token.
* Sensitive credentials.
* Challenge answers.
* Personally identifiable information contained within the lab database.

The focus of this report is understanding the vulnerability lifecycle, practicing ethical security testing, and documenting findings using a professional penetration testing methodology.

---

