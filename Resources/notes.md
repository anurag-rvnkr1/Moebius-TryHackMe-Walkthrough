# 📝 Moebius — Technical Notes & Learning Journal

> Technical study notes derived from the authorized TryHackMe Moebius assessment.

---

# Key Concepts

## 1. SQL Injection

The `short_tag` parameter was incorporated into a SQL query without safe parameter binding.

### Indicators

* Database syntax errors caused by crafted input.
* Boolean behavior changes.
* UNION-based output.
* sqlmap confirmation.

### Lesson

Prepared statements should be the primary defense. Input filtering is not a substitute for safe query construction.

---

## 2. Character-Blacklist Bypass

The application rejected `/` and `;`:

```php
preg_match('/[\/;]/', $_GET['short_tag'])
```

The attack chain avoided the direct slash character by representing filesystem paths as hexadecimal SQL literals.

### Lesson

Security controls should validate the intended operation, not attempt to blacklist a handful of characters.

---

## 3. PHP Stream Wrappers

The `php://filter` wrapper was used to transform file contents before they were returned.

Example:

```text
php://filter/read=convert.base64-encode/resource=/var/www/html/index.php
```

This is particularly useful during authorized testing because PHP source can otherwise be interpreted by the web server instead of displayed.

---

## 4. HMAC-Protected File Retrieval

The application generated:

```php
hash_hmac('sha256', $path, $SECRET_KEY)
```

This demonstrates an important security design distinction:

* An HMAC can protect the integrity of a path value.
* It does not make an arbitrary path safe if an attacker can obtain the signing secret.

---

## 5. PHP Filter Chains

PHP stream filters can transform data through multiple stages. In vulnerable applications, filter chains can become part of a larger code-execution chain.

The supplied material used a filter-chain generator around:

```php
<?php eval($_REQUEST[0]);?>
```

The resulting execution path encountered disabled PHP functions, demonstrating the importance of understanding runtime restrictions rather than assuming every command-execution function is available.

---

## 6. `LD_PRELOAD`

`LD_PRELOAD` allows a shared object to be loaded before normal shared libraries when a suitable dynamically linked process starts.

In the challenge, the supplied technique used a compiled shared object and a PHP-triggered `mail()` invocation.

### Defensive lesson

Web applications should not be able to freely control environment variables that influence privileged or security-sensitive process execution.

---

## 7. Container Capabilities

The compromised process exposed:

```bash
grep CapEff /proc/self/status
```

with a very broad effective capability mask.

### Lesson

Container security depends on the privileges granted to the container. A shell inside a container is not automatically harmless if the container can interact with host devices.

---

## 8. `privileged: true`

The recovered Compose configuration contained:

```yaml
privileged: true
```

This setting is inappropriate for a normal public-facing web application unless there is a documented operational requirement and compensating controls.

---

## 9. Host Filesystem Mounting

The supplied post-exploitation sequence mounted:

```text
/dev/nvme0n1p1
```

under:

```text
/mnt/tmp3
```

The mounted filesystem contained a complete Linux root hierarchy.

### Lesson

Host block devices must be treated as highly sensitive resources and should not be reachable by application containers.

---

# Enumeration Checklist

* [x] Full TCP scan
* [x] Service/version enumeration
* [x] Web endpoint analysis
* [x] SQL injection validation
* [x] Database enumeration
* [x] File disclosure
* [x] PHP source recovery
* [x] Application secret analysis
* [x] Shell context verification
* [x] Container capability review
* [x] Docker deployment review
* [x] Host filesystem access analysis
* [x] Database configuration review

---

# Commands Worth Remembering

## Network Enumeration

```bash
nmap -sS -sV -p- -A TARGET
```

## SQL Injection Validation

```bash
sqlmap "http://TARGET/album.php?short_tag=cute"
```

## Web Content Discovery

```bash
ffuf -u http://TARGET/FUZZ \
     -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt \
     -e .php -ac
```

## Capability Review

```bash
grep CapEff /proc/self/status
```

## Docker Enumeration

```bash
docker ps -a
```

## Database Enumeration

```sql
SHOW DATABASES;
USE secret;
SHOW TABLES;
```

---

# Personal Takeaways

* Follow data flows, not just individual vulnerabilities.
* Source disclosure can expose the assumptions that make an exploit chain possible.
* Blacklists should never be treated as a primary security boundary.
* Container privileges must be audited as part of application security.
* Deployment files can become a credential source after host compromise.
* Good penetration testing connects technical exploitation with root cause and remediation.
