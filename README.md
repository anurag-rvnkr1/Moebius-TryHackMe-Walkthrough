# 🌀 Moebius — TryHackMe CTF Walkthrough

<p align="center">
  <img src="docs/assets/banner.png" alt="Moebius TryHackMe Room" width="100%">
</p>

<p align="center">
  <a href="https://tryhackme.com/room/moebius">
    <img src="https://img.shields.io/badge/TryHackMe-Moebius-red?style=for-the-badge&logo=tryhackme" />
  </a>
  <img src="https://img.shields.io/badge/Difficulty-Not%20Specified-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Platform-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" />
  <img src="https://img.shields.io/badge/Focus-Web%20%2B%20Container%20Security-blue?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/SQL%20Injection-Exploitation-6f42c1?style=flat-square" />
  <img src="https://img.shields.io/badge/PHP%20Filter%20Chain-RCE-critical?style=flat-square" />
  <img src="https://img.shields.io/badge/Docker-Container%20Escape-success?style=flat-square" />
  <img src="https://img.shields.io/badge/Documentation-Portfolio%20Project-0A66C2?style=flat-square" />
</p>

---

## 📌 Overview

**Moebius** is a TryHackMe Linux web-security challenge built around a chained compromise. The attack begins with a SQL injection in the `short_tag` parameter, develops into local file disclosure and PHP source-code analysis, reaches application-level code execution, and ultimately crosses a weak Docker isolation boundary.

This repository documents the compromise as a **professional penetration-testing case study** rather than a simple answer sheet. The focus is on attack-surface analysis, evidence, exploitation reasoning, post-exploitation, container security, and defensive remediation.

> **Purpose:** Demonstrate practical web exploitation, Linux enumeration, PHP application analysis, container security, and security reporting within an authorized TryHackMe laboratory.

---

# 🎯 Objectives

This walkthrough demonstrates how to:

* Perform full TCP reconnaissance and service identification.
* Validate SQL injection in the `album.php?short_tag=` parameter.
* Enumerate the application database with `sqlmap`.
* Analyze the application's file-retrieval mechanism.
* Bypass the application's `/` and `;` character filter using hexadecimal representation.
* Retrieve local files and PHP source through `php://filter`.
* Recover the application secret required for valid HMAC-protected image URLs.
* Use a PHP filter-chain execution technique and the referenced `LD_PRELOAD`/Chankro method to obtain code execution.
* Analyze the resulting privileged Docker container.
* Mount the host filesystem from the privileged container.
* Inspect the host-side Docker deployment and protected database.

---

# 🧠 Skills Demonstrated

| Domain | Techniques |
| --- | --- |
| **Reconnaissance** | Nmap SYN scan, version detection, OS fingerprinting |
| **Web Enumeration** | Parameter analysis, FFUF, source review |
| **Web Exploitation** | SQL injection, UNION-based manipulation, filter bypass |
| **File Disclosure** | `php://filter`, local file retrieval |
| **PHP Security** | Stream wrappers, filter chains, `LD_PRELOAD` execution path |
| **Container Security** | Docker enumeration, privileged container analysis, host filesystem access |
| **Post-Exploitation** | Shell verification, filesystem inspection, deployment analysis |
| **Reporting** | Attack-chain analysis, findings, remediation, MITRE ATT&CK mapping |

---

# ⚙️ Lab Information

| Property | Value |
| --- | --- |
| Platform | TryHackMe |
| Room | Moebius |
| Room URL | https://tryhackme.com/room/moebius |
| Operating System | Linux / containerized web application |
| Initial Target IP | `10.10.114.39` |
| Web Server | Apache HTTP Server 2.4.62 (Debian) |
| SSH | OpenSSH 8.9p1 |
| Primary Attack Surface | `album.php?short_tag=` |
| Environment | Authorized Training Lab |

> **IP note:** The supplied material uses `10.10.114.39` during reconnaissance and `10.10.135.217` in later exploitation examples. Both are retained only where they are present in the supplied evidence; `TARGET` is used where the later session address cannot be independently established.

> **Difficulty note:** The supplied source material does not state a difficulty rating, so none is inferred.

---

# 🛠️ Tools Used

| Tool / Technology | Purpose |
| --- | --- |
| **Nmap** | Network reconnaissance and service enumeration |
| **sqlmap** | SQL injection validation and database enumeration |
| **FFUF** | Web content discovery |
| **Python** | File-retrieval automation |
| **PHP Filter Chain Generator** | Construction of PHP filter chains |
| **Chankro technique** | `LD_PRELOAD`-based execution path referenced by the source material |
| **curl** | HTTP retrieval and payload delivery |
| **Docker** | Container and host-boundary enumeration |
| **MariaDB/MySQL client** | Database inspection |

---

# 🔍 Attack Methodology

```text
Reconnaissance
      │
      ▼
Service Enumeration
      │
      ▼
Web Application Analysis
      │
      ▼
SQL Injection
      │
      ▼
Filter Bypass + File Disclosure
      │
      ▼
PHP Source / Secret Recovery
      │
      ▼
PHP Filter Chain RCE
      │
      ▼
Container Foothold
      │
      ▼
Privileged Container Analysis
      │
      ▼
Host Filesystem Mount
      │
      ▼
Host Deployment Enumeration
      │
      ▼
Protected Database Access
      │
      ▼
FINAL OBJECTIVE
```

The detailed case study explains the evidence and reasoning behind each transition.

---

# 🗂️ Repository Structure

```text
Moebius-TryHackMe-Walkthrough/
│
├── README.md
├── _config.yml
│
├── Documentation/
│   └── THM_Moebius_Documentation.md
│
├── Resources/
│   ├── notes.md
│   ├── payloads.md
│   ├── tools.md
│   ├── references.md
│   └── remediation.md
│
├── Screenshots/
│   ├── figure-1-room-overview.png
│   ├── figure-2-reverse-shell.png
│   ├── figure-3-php-filter-chain-error.png
│   ├── figure-4-lfi-passwd.png
│   ├── figure-5-union-query-source.png
│   ├── figure-6-sqli-union-validation.png
│   ├── figure-7-image-file-retrieval.png
│   ├── figure-8-sqlmap-enumeration.png
│   ├── figure-9-sqli-error-validation.png
│   ├── figure-10-web-application.png
│   └── figure-11-container-environment.png
│
├── docs/
│   ├── index.md
│   └── assets/
│       ├── banner.png
│       ├── figure-*.png
│       └── css/
│           └── custom.scss
│
└── .github/
    └── workflows/
        └── pages.yml
```

---

# 🕵️ Attack Surface Summary

| Attack Surface | Observation |
| --- | --- |
| **22/tcp** | OpenSSH 8.9p1 |
| **80/tcp** | Apache 2.4.62 serving the Image Grid application |
| **`album.php`** | `short_tag` is injectable |
| **`image.php`** | Accepts `hash` and `path` and retrieves application image files |
| **PHP source** | Recoverable through the file-read primitive |
| **Docker web service** | Recovered Compose configuration uses `privileged: true` |
| **Host filesystem** | Accessible from the privileged container through a host block device |

---

# 💥 Exploitation Highlights

## Phase 1 — Reconnaissance

Nmap identified SSH and HTTP. The web application was the primary candidate for further analysis.

## Phase 2 — SQL Injection

`short_tag` was confirmed as injectable and supported UNION-based manipulation. Database enumeration identified the `web` database and its `albums` and `images` tables.

## Phase 3 — File Disclosure

The application filtered `/` and `;`. Hexadecimal representation was used to express filesystem paths without sending the filtered slash character directly. This enabled retrieval of `/etc/passwd` and PHP source.

## Phase 4 — PHP Source Analysis and RCE

Source disclosure revealed the HMAC construction used by `image.php` and exposed application configuration. The source material then used a PHP filter-chain technique and an `LD_PRELOAD`/Chankro execution path to obtain a shell.

## Phase 5 — Container-to-Host Boundary Failure

The resulting shell ran as `www-data` inside a container. Capability enumeration and the privileged Docker configuration showed that the container could access a host block device and mount the host filesystem.

## Phase 6 — Protected Data Access

The host-side Docker deployment exposed the database configuration and a separate `secret` database. The final challenge value is intentionally redacted from this repository.

---

# 🛡️ Security Findings

| Finding | Assessment Severity |
| --- | --- |
| SQL Injection in `short_tag` | 🔴 High |
| Arbitrary local file disclosure | 🔴 High |
| Application secret exposed through source/configuration | 🟠 High |
| PHP filter-chain / application RCE path | 🔴 Critical |
| Privileged Docker container | 🔴 Critical |
| Host filesystem exposed from container | 🔴 Critical |
| Plaintext database credentials in deployment configuration | 🟠 High |

> These are portfolio assessment ratings derived from the supplied attack chain. They are not official TryHackMe ratings or CVSS scores.

---

# 🧬 MITRE ATT&CK Mapping

| Tactic | Technique | Relevance |
| --- | --- | --- |
| Initial Access | **T1190 — Exploit Public-Facing Application** | SQL injection against the exposed application |
| Credential Access | **T1552.001 — Credentials In Files** | Database credentials recovered from deployment configuration |
| Discovery | **T1083 — File and Directory Discovery** | Container and host filesystem enumeration |
| Execution | **T1059.004 — Unix Shell** | Shell execution after application compromise |
| Privilege Escalation / Defense Evasion | **T1611 — Escape to Host** | Privileged container plus host filesystem access |

---

# 📖 Documentation

| Document | Description |
| --- | --- |
| **THM_Moebius_Documentation.md** | Complete technical walkthrough and security analysis |
| **docs/index.md** | GitHub Pages case-study presentation |
| **Resources/** | Notes, payload references, tools, references, and remediation |

---

# 🖼️ Evidence Policy

The repository uses the supplied Moebius screenshots as technical evidence. Conceptual descriptions are explicitly separated from observed output.

Screenshots are placed beside the corresponding attack stage rather than collected at the end of the report.

No screenshot was fabricated to represent command output that was not supplied.

---

# 🚩 Flag Policy

The actual TryHackMe challenge flag is intentionally redacted.

```text
Final challenge value → [FLAG REDACTED]
```

The flag is not included in filenames, captions, alt text, diagrams, metadata, README content, or GitHub Pages content.

---

# 📚 Key Learning Outcomes

* SQL injection can become a pivot into file disclosure and application-source analysis.
* Character-based filters are not reliable security controls when alternate encodings remain accepted.
* PHP stream wrappers can expose source code and, in unsafe designs, contribute to an execution chain.
* Application secrets must never be embedded in web-readable source or insecure deployment files.
* `privileged: true` materially weakens container isolation.
* Host block devices must never be unnecessarily reachable from an application container.
* Container deployment configuration is part of the security boundary and must be reviewed like application code.

---

# 🔐 Remediation Summary

| Vulnerability | Recommended Mitigation |
| --- | --- |
| SQL Injection | Use parameterized queries and strict server-side input handling. |
| Arbitrary file retrieval | Use application-owned image identifiers and never accept raw filesystem paths. |
| Source/config disclosure | Keep secrets outside web-accessible application paths and use a secret-management mechanism. |
| PHP filter-chain/RCE path | Eliminate attacker-controlled file paths and harden PHP stream-wrapper usage. |
| Privileged container | Remove `privileged: true` unless operationally unavoidable; apply least privilege. |
| Host device access | Do not expose host block devices or host filesystems to application containers. |
| Plaintext database credentials | Use secrets management, restrict access, and rotate exposed credentials. |

---

# 🌐 GitHub Pages

The repository follows the portfolio structure established by the Airplane reference project.

The Pages workflow builds the documentation from `./docs` and deploys it using the official GitHub Pages actions.

---

# ⚠️ Disclaimer

This repository documents exploitation techniques performed **only inside an authorized TryHackMe laboratory**.

The material is provided for:

* Cybersecurity education.
* Defensive security learning.
* Capture The Flag documentation.
* Authorized penetration-testing practice.

Do not use these techniques against systems without explicit authorization.

---

# 👨‍💻 Author

## **Anurag Ravankar**

Cybersecurity Enthusiast • Penetration Testing • SOC • Linux Security

* Linux Security
* Web Application Security
* Capture The Flag (CTF) Writeups
* TryHackMe Documentation
* Security Research
