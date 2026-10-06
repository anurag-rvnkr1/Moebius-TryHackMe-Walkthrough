# 🛠️ Moebius — Tool Reference

This document explains the role of each tool in the assessment.

---

# Tool Stack

| Tool | Role |
| --- | --- |
| Nmap | Reconnaissance and service enumeration |
| sqlmap | SQL injection validation and database enumeration |
| FFUF | Web content discovery |
| Python | Custom file retrieval |
| PHP Filter Chain Generator | PHP filter-chain construction |
| curl | HTTP retrieval and payload delivery |
| GCC | Shared-object compilation |
| Netcat / rlwrap | Reverse-shell listener |
| Docker | Container and host-boundary enumeration |
| MariaDB client | Database inspection |

---

# Nmap

## Purpose

* Discover exposed ports.
* Identify services.
* Determine versions.
* Gather OS-level information.

### Why it mattered

The HTTP service became the main attack surface after reconnaissance.

---

# sqlmap

## Purpose

* Confirm SQL injection.
* Identify supported injection techniques.
* Enumerate databases.
* Enumerate tables.

### Why it mattered

Manual testing showed SQL error behavior; sqlmap provided systematic validation and database enumeration.

---

# FFUF

## Purpose

Discover common web application paths and PHP endpoints.

### Why it mattered

The application surface included `index.php`, `image.php`, and `album.php`. Parameter analysis then became more important than directory discovery.

---

# Python

## Purpose

The supplied source material used Python to automate:

* Path encoding.
* HTTP requests.
* Retrieval of the generated image URL.
* Saving returned file content.

### Why it mattered

Automation reduced repeated manual requests while testing the file-disclosure primitive.

---

# PHP Filter Chain Generator

## Purpose

Construct a PHP stream-filter chain for a controlled code-execution payload.

### Why it mattered

The application exposed PHP stream-wrapper functionality through the file-processing path.

---

# curl

## Purpose

Retrieve a payload from an HTTP server and save it on the target.

### Why it mattered

The supplied `LD_PRELOAD` technique used HTTP retrieval to place a shared object in `/tmp`.

---

# GCC

## Purpose

Compile the supplied shared-object payload.

```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

---

# Netcat

## Purpose

Receive the reverse shell.

```bash
rlwrap nc -lvnp 4444
```

---

# Docker

## Purpose

Enumerate the containerized environment and inspect the recovered deployment.

```bash
docker ps -a
```

The recovered Compose configuration was the key evidence demonstrating the privileged web container.

---

# MariaDB Client

## Purpose

Inspect the database after host-boundary access.

The supplied workflow identified the `secret` database and its `secrets` table.

---

# Defensive Perspective

| Tool | Defensive Use |
| --- | --- |
| Nmap | External attack-surface validation |
| sqlmap | Authorized injection testing |
| FFUF | Web asset inventory |
| Docker | Container configuration auditing |
| MariaDB client | Database access validation |
| Netcat | Controlled shell testing in a lab |

---

# Recommended Workflow

1. Map the network attack surface.
2. Enumerate application endpoints.
3. Validate input handling.
4. Determine the impact of the discovered primitive.
5. Review application source when legitimately obtainable.
6. Analyze the execution context.
7. Audit container privileges.
8. Review deployment secrets.
9. Document root cause and remediation.
