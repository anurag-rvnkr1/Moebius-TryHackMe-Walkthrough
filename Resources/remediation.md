# 🛡️ Moebius — Security Findings & Remediation Guide

This document summarizes the security weaknesses demonstrated by the Moebius attack chain.

---

# Executive Summary

The compromise depended on multiple weaknesses rather than a single isolated vulnerability:

* SQL injection.
* Arbitrary local file disclosure.
* Application-secret exposure.
* Dangerous PHP execution behavior.
* Privileged container deployment.
* Host block-device exposure.
* Plaintext database credentials in deployment configuration.

Together, these weaknesses allowed a web application compromise to progress into host-level filesystem access and protected database access.

---

# Finding 1 — SQL Injection

**Risk:** High

### Root Cause

User-controlled input was concatenated into SQL.

### Impact

Attackers could manipulate database queries and enumerate database structure.

### Remediation

* Use prepared statements.
* Bind parameters through PDO.
* Do not build SQL with string concatenation.
* Return generic database errors.

---

# Finding 2 — Arbitrary File Disclosure

**Risk:** High

### Root Cause

The application accepted a filesystem path and relied on a character blacklist.

### Impact

Sensitive operating-system and application files became readable.

### Remediation

* Replace raw paths with application-owned IDs.
* Enforce allowlists.
* Canonicalize and validate paths.
* Keep sensitive files outside application-readable locations.

---

# Finding 3 — Application Secret Exposure

**Risk:** High

### Root Cause

The application secret was stored in application configuration reachable through source disclosure.

### Impact

An attacker could reproduce HMAC values used by the image retrieval mechanism.

### Remediation

* Use a secret manager.
* Keep secrets outside the web root.
* Restrict configuration permissions.
* Rotate secrets after exposure.

---

# Finding 4 — PHP Execution Chain

**Risk:** Critical

### Root Cause

Attacker-controlled file processing reached PHP stream-filter behavior and an execution technique based on `LD_PRELOAD`.

### Impact

Remote code execution and a reverse shell were obtained.

### Remediation

* Remove arbitrary file path handling.
* Avoid dynamic evaluation.
* Disable unnecessary dangerous runtime functionality.
* Restrict environment-variable influence over child processes.
* Apply least privilege to the web process.

---

# Finding 5 — Privileged Docker Deployment

**Risk:** Critical

### Root Cause

The recovered Compose file used:

```yaml
privileged: true
```

### Impact

The container received broad host-level privileges.

### Remediation

* Remove privileged mode.
* Drop capabilities.
* Restrict devices.
* Apply seccomp/AppArmor/SELinux controls.
* Run the application as a dedicated non-root identity.

---

# Finding 6 — Host Device Exposure

**Risk:** Critical

### Root Cause

The compromised container could access a host block device.

### Impact

The host filesystem became accessible from the container.

### Remediation

* Block host device access.
* Apply device cgroup restrictions.
* Monitor mount operations.
* Treat application containers as untrusted workloads.

---

# Finding 7 — Plaintext Database Credentials

**Risk:** High

### Root Cause

Database credentials were present in deployment configuration.

### Impact

Host compromise enabled database authentication and access to protected data.

### Remediation

* Use container/orchestrator secrets.
* Rotate credentials.
* Restrict environment-file permissions.
* Avoid embedding root database credentials in application deployments.

---

# Blue-Team Detection Opportunities

| Offensive Activity | Defensive Signal |
| --- | --- |
| SQL injection | Repeated SQL syntax errors and anomalous parameters |
| File disclosure | Unusual web-process filesystem reads |
| PHP filter abuse | Requests containing `php://` wrappers |
| External process execution | Web worker spawning unexpected binaries |
| Reverse shell | Unexpected outbound connection from web service |
| Container escape | `mount` operations or device access from containers |
| Database access | Authentication from unexpected process/container context |

---

# Linux Hardening Checklist

* [ ] Remove unnecessary SUID/capabilities.
* [ ] Avoid privileged containers.
* [ ] Restrict host devices.
* [ ] Apply seccomp/AppArmor/SELinux controls.
* [ ] Review container identities.
* [ ] Restrict configuration-file permissions.
* [ ] Rotate exposed credentials.
* [ ] Monitor mount operations.

---

# Web Application Checklist

* [ ] Parameterized SQL everywhere.
* [ ] No raw filesystem paths from user input.
* [ ] Allowlist application resources.
* [ ] Keep secrets outside the web root.
* [ ] Disable unnecessary PHP functionality.
* [ ] Return generic database errors.
* [ ] Monitor suspicious request patterns.

---

# Conclusion

Moebius demonstrates the importance of layered security. A SQL injection was dangerous because it enabled a broader file-disclosure chain; the container was dangerous because it was privileged; and the database became reachable because deployment credentials were exposed after the host boundary was crossed.

Defensive controls should therefore be designed so that failure of one layer does not automatically produce full environment compromise.
