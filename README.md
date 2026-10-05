# 🌀 Moebius — TryHackMe CTF Walkthrough

<p align="center">
  <img src="docs/assets/banner.png" alt="Moebius TryHackMe Banner" width="100%">
</p>

<p align="center">
  <a href="https://tryhackme.com/room/moebius">
    <img src="https://img.shields.io/badge/TryHackMe-Moebius-red?style=for-the-badge&logo=tryhackme" />
  </a>
  <img src="https://img.shields.io/badge/Difficulty-Not%20Specified-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Platform-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" />
  <img src="https://img.shields.io/badge/Focus-Web%20Exploitation-blue?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/SQL%20Injection-Exploitation-6f42c1?style=flat-square"/>
  <img src="https://img.shields.io/badge/PHP%20Filter%20Chain-RCE-critical?style=flat-square"/>
  <img src="https://img.shields.io/badge/Docker-Container%20Escape-success?style=flat-square"/>
  <img src="https://img.shields.io/badge/Documentation-Portfolio%20Project-0A66C2?style=flat-square"/>
</p>

---

## 📌 Overview

**Moebius** is a Linux-based Capture The Flag room on **TryHackMe** that demonstrates how a web application vulnerability can be chained with application-source disclosure, PHP stream-wrapper abuse, remote code execution, and an unsafe container deployment to reach data outside the intended application boundary.

This repository presents the room as a **professional penetration-testing case study** rather than a simple answer sheet. The emphasis is on attack-surface analysis, vulnerability validation, exploitation reasoning, evidence handling, and defensive remediation.

> **Purpose:** Demonstrate practical web security, Linux post-exploitation, PHP application analysis, container security, and professional security documentation within an authorized TryHackMe laboratory.

---

# 🎯 Objectives

This walkthrough demonstrates how to:

* Perform full TCP reconnaissance with Nmap.
* Identify and validate SQL injection in the `short_tag` parameter.
* Enumerate the application database structure.
* Bypass the application's `/` and `;` input filter using hexadecimal representation.
* Retrieve local files through the application's image-loading functionality.
* Use `php://filter` to disclose PHP source code.
* Recover application secrets required to construct valid image URLs.
* Chain PHP stream filters toward application-level code execution.
* Use the documented Chankro/`LD_PRELOAD` technique described in the source material to obtain code execution.
* Enumerate a privileged Docker container and mount the host filesystem.
* Access the host-side challenge data through the container deployment weakness.

---

# 🧠 Skills Demonstrated

| Domain | Techniques |
| --- | --- |
| **Reconnaissance** | Nmap SYN scan, version detection, OS fingerprinting |
| **Web Enumeration** | Parameter analysis, FFUF, PHP source review |
| **Web Exploitation** | SQL injection, UNION-based manipulation, filter bypass |
| **File Disclosure** | `php://filter`, application file retrieval |
| **PHP Security** | Stream wrappers, filter chains, `LD_PRELOAD` execution path |
| **Container Security** | Docker enumeration, privileged container analysis, host filesystem mounting |
| **Post-Exploitation** | Filesystem inspection, Docker and database enumeration |
| **Reporting** | Attack-chain analysis, security findings, remediation |

---

# ⚙️ Lab Information

| Property | Value |
| --- | --- |
| Platform | TryHackMe |
| Room | Moebius |
| Operating System | Linux / containerized web application |
| Target IP | `10.10.114.39` in the supplied reconnaissance evidence |
| Web Server | Apache HTTP Server 2.4.62 (Debian) |
| SSH | OpenSSH 8.9p1 |
| Primary Attack Surface | `album.php?short_tag=` |
| Environment | Authorized Training Lab |

> **Difficulty:** The supplied source material does not state a room difficulty. It is therefore not inferred here.
>
> **IP note:** The source material contains different lab IPs at different stages (`10.10.114.39` during initial reconnaissance and `10.10.135.217` in later exploitation examples). This documentation preserves the initial scan value and uses `TARGET` when the exact later session IP is not independently verifiable.

---

# 🛠️ Tools Used

| Tool / Technology | Purpose |
| --- | --- |
| **Nmap** | Network reconnaissance and service enumeration |
| **sqlmap** | SQL injection validation and database enumeration |
| **FFUF** | Web content discovery |
| **Python** | Custom file-retrieval automation |
| **PHP Filter Chain Generator** | Construction of PHP filter chains |
| **Chankro** | `LD_PRELOAD`-based execution technique referenced by the source material |
| **curl** | HTTP retrieval and payload delivery |
| **Docker** | Container and host-boundary enumeration |
| **MariaDB/MySQL client** | Database inspection |

---

# 🔍 Attack Methodology

The assessment followed a structured penetration-testing workflow in which each stage was driven by evidence obtained during the preceding phase.

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
Protected Database Access
      │
      ▼
FINAL OBJECTIVE
```

A visual version of this methodology is included in the GitHub Pages documentation as a **conceptual diagram**, not as evidence.

---

# 🗂️ Repository Structure

```text
Moebius-TryHackMe-Walkthrough/
│
├── README.md
├── _config.yml
│
├── Documentation/
│   ├── THM_Moebius_Documentation.md
│   └── THM_Moebius_Report.pdf
│
├── Resources/
│   ├── notes.md
│   ├── payloads.md
│   ├── tools.md
│   ├── references.md
│   └── remediation.md
│
├── Screenshots/
│   └── README.md  # Evidence policy; no fabricated screenshots
│
├── docs/
│   ├── index.md
│   └── assets/
│       ├── banner.png
│       ├── attack-chain.png
│       └── css/
│           └── custom.scss
│
└── .github/
    └── workflows/
        └── pages.yml
```

The `Screenshots/` directory is intentionally not populated with fabricated terminal captures. The source material supplied for this build contained textual command/output evidence but no verifiable Moebius screenshot set. The documentation therefore distinguishes source-derived evidence from the conceptual visuals.

---

# 🕵️ Attack Surface Summary

| Attack Surface | Observation |
| --- | --- |
| **22/tcp** | OpenSSH 8.9p1 |
| **80/tcp** | Apache 2.4.62 serving the Image Grid application |
| **`album.php`** | `short_tag` parameter vulnerable to SQL injection |
| **`image.php`** | Accepts `hash` and `path`, enabling controlled file retrieval when a valid HMAC is supplied |
| **Docker web service** | Deployed as a privileged container according to the recovered Compose configuration |
| **Host filesystem** | Mounted from inside the privileged container, breaking the intended container isolation boundary |

---

# 💥 Exploitation Highlights

## Phase 1 — Reconnaissance

A full TCP scan identified SSH and HTTP as the exposed services. The HTTP service became the primary focus because it exposed an application with a user-controlled `short_tag` parameter.

## Phase 2 — SQL Injection

`album.php?short_tag=` accepted SQL syntax and was confirmed by sqlmap to support multiple injection techniques, including UNION-based manipulation.

## Phase 3 — File Disclosure

The application filtered `/` and `;`. Hexadecimal encoding was used to represent filesystem paths without placing the blocked slash character directly into the request. This enabled retrieval of `/etc/passwd` and, more importantly, PHP source code through `php://filter`.

## Phase 4 — Application-Level RCE

Source review exposed the HMAC construction used by `image.php`. With the application secret available from the disclosed PHP configuration, valid image URLs could be constructed. The source material then used a PHP filter-chain technique and Chankro's `LD_PRELOAD` approach to reach code execution.

## Phase 5 — Container-to-Host Boundary Failure

The resulting shell was inside a Docker container. Enumeration showed a privileged web container and access to the host's block device. Mounting the host filesystem exposed the host operating system and the challenge deployment directory.

## Phase 6 — Protected Data Access

The host-side Compose and database configuration exposed the MariaDB deployment. The secret database contained the final challenge value, which is intentionally redacted from this public repository.

---

# 🛡️ Security Findings

| Finding | Assessment Severity |
| --- | --- |
| SQL Injection in `short_tag` | 🔴 High |
| Arbitrary local file disclosure | 🔴 High |
| Application secret disclosed through source/configuration | 🟠 High |
| PHP filter-chain / application RCE path | 🔴 Critical |
| Privileged Docker container | 🔴 Critical |
| Host filesystem exposed from container | 🔴 Critical |
| Plaintext database credentials in deployment configuration | 🟠 High |

These are **portfolio assessment severities**, not an official TryHackMe scoring system or a CVSS calculation.

---

# 🧬 MITRE ATT&CK Mapping

| Tactic | Technique | Relevance |
| --- | --- | --- |
| Initial Access | **T1190 — Exploit Public-Facing Application** | SQL injection against the exposed web application |
| Credential Access | **T1552.001 — Credentials In Files** | Database credentials recovered from configuration |
| Discovery | **T1083 — File and Directory Discovery** | Host/container filesystem enumeration |
| Execution | **T1059.004 — Unix Shell** | Shell execution after application compromise |
| Privilege Escalation / Defense Evasion | **T1611 — Escape to Host** | Privileged container plus host filesystem access |

The mapping is limited to techniques directly supported by the supplied attack chain.

---

# 📖 Documentation

| Document | Description |
| --- | --- |
| **THM_Moebius_Documentation.md** | Complete technical walkthrough and security analysis |
| **THM_Moebius_Report.pdf** | Portfolio-style penetration-testing report |
| **docs/index.md** | GitHub Pages case-study presentation |
| **Resources/** | Notes, payload references, tools, references, and remediation |

---

# 🖼️ Evidence Policy

No verifiable Moebius screenshots were available in the current repository/source-material set used for this build. Accordingly:

* No terminal screenshot was fabricated.
* No fake exploit output was generated.
* No screenshot was renamed from another CTF.
* The supplied textual command output is preserved as source-derived evidence in the technical documentation.
* `banner.png` and `attack-chain.png` are explicitly **conceptual portfolio visuals**, not attack evidence.

---

# 🚩 Flag Policy

The actual TryHackMe flag is intentionally **redacted**.

```text
Final challenge value → [FLAG REDACTED]
```

The flag is not included in filenames, metadata, screenshots, diagrams, README content, or GitHub Pages content.

---

# 📚 Key Learning Outcomes

* SQL injection should be investigated beyond simple authentication bypasses; UNION-based injection can become a bridge into file disclosure and source-code analysis.
* Input filters that block individual characters are weak when the underlying application accepts alternate encodings.
* PHP stream wrappers can turn a file-read primitive into source disclosure and, in vulnerable application designs, into a more powerful execution chain.
* Application secrets should never be hardcoded into web-accessible source or deployment files.
* A privileged Docker container materially weakens container isolation and can enable host compromise when host devices/filesystems are reachable.
* Container deployment configuration must be treated as security-sensitive infrastructure.

---

# 🔐 Remediation Summary

| Vulnerability | Recommended Mitigation |
| --- | --- |
| SQL Injection | Use parameterized queries and strict server-side input handling. |
| Arbitrary file retrieval | Use an allowlist of application-owned image identifiers; never accept raw filesystem paths. |
| Source/config disclosure | Keep secrets outside web-readable application paths and use a dedicated secret-management mechanism. |
| PHP filter-chain/RCE path | Disable unnecessary stream-wrapper functionality, harden PHP, and eliminate attacker-controlled file paths. |
| Privileged container | Remove `privileged: true` unless operationally unavoidable; apply least privilege. |
| Host device access | Do not expose host block devices or host filesystems to application containers. |
| Plaintext database credentials | Use secrets management, restricted environment files, and rotated credentials. |

---

# 🌐 GitHub Pages

The repository follows the Airplane portfolio structure and includes a GitHub Pages site under `docs/`.

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

---

<p align="center">
  ⭐ Professional cybersecurity documentation focused on methodology, evidence, and defensive understanding.
</p>
