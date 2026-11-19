# Log Analysis & SIEM (Defensive)

> **TL;DR:** A SIEM (Splunk, Elastic/ELK, Wazuh, Microsoft Sentinel) aggregates logs from endpoints, firewalls, and apps into one searchable timeline. Know the high-value **Linux logs**, **Windows Security & Sysmon Event IDs**, and how to write basic SIEM correlation queries.

---

## Core Concepts & Mental Model

### High-Value Log Sources & Event IDs
| Platform | Log / Event ID | What It Records & Why Defenders Care |
|----------|----------------|--------------------------------------|
| **Linux** | `/var/log/auth.log` (Debian) / `/var/log/secure` (RHEL) | SSH logins, failed passwords, `sudo` usage, new user creation |
| **Linux** | `/var/log/apache2/access.log` or `nginx/access.log` | HTTP methods, URIs, status codes (`404` fuzzing spikes), User-Agents (`sqlmap`) |
| **Windows Security** | `4624` / `4625` | Successful logon / **Failed logon** (brute-force & password spraying) |
| **Windows Security** | `4688` | Process Creation (enable command-line auditing in GPO!) |
| **Windows Security** | `4720` / `4732` | New user account created / User added to local security group (e.g., Admins) |
| **Windows Security** | `1102` | Security audit log was **cleared** (major red flag!) |
| **Sysmon** | `Event ID 1` | Process Creation (includes parent process, full command line, and SHA-256 hash) |
| **Sysmon** | `Event ID 3` | Network Connection initiated by a process |
| **Sysmon** | `Event ID 11` | File Created |
| **PowerShell** | `Event ID 4104` | Script Block Logging (captures decoded/obfuscated PowerShell scripts) |

---

## Quick-Reference Query Cheatsheet

### 1. Linux CLI Log Triage (`grep` / `awk` / `journalctl`)
| Command | Purpose |
|---------|---------|
| `grep "Accepted" /var/log/auth.log` | List all successful SSH logins |
| `grep "Failed password" /var/log/auth.log \| awk '{print $(NF-3)}' \| sort \| uniq -c \| sort -nr` | Rank top attacking IPs by failed SSH attempts |
| `grep -E "404\|403" /var/log/nginx/access.log \| awk '{print $1}' \| sort \| uniq -c \| sort -nr` | Spot directory fuzzers generating `404`/`403` spikes |
| `journalctl -u ssh --since "1 hour ago"` | Query systemd journal for SSH events in the last hour |

### 2. Splunk SPL (Search Processing Language) Examples
```spl
# Detect potential brute-force ( > 10 failed logons in 5 mins per IP )
index=windows EventCode=4625
| bin _time span=5m
| stats count by _time, Source_Network_Address, Account_Name
| where count > 10
```

```spl
# Detect suspicious Office -> Shell spawning (Macro / Phishing)
index=sysmon EventCode=1 (ParentImage="*\\WINWORD.EXE" OR ParentImage="*\\EXCEL.EXE")
  (Image="*\\cmd.exe" OR Image="*\\powershell.exe")
| table _time, Computer, User, ParentImage, Image, CommandLine
```

---

## Common Pitfalls & Defensive Notes
- **Time Zone Mismatches:** Always normalize all logs to **UTC** before ingesting into a SIEM; correlating an attack across local timezones causes missed timelines.
- **Logging Without Command-Line Args:** Default Windows `4688` does not log command-line arguments unless enabled via GPO (or Sysmon is deployed). Always deploy **Sysmon** (with a community config like SwiftOnSecurity or Olaf Hartong) for real endpoint visibility.

---

## Hands-On Lab: Hunting Directory Fuzzing & SQLi in Web Access Logs on Kali

**Goal:** Create a realistic Nginx/Apache access log containing normal traffic, `ffuf` 404 bursts, and a `sqlmap` probe, then write CLI hunting one-liners to isolate the attacker.

### Step 1: Generate Sample Web Logs
```bash
mkdir -p /tmp/log_lab
cat << 'EOF' > /tmp/log_lab/access.log
10.0.0.5 - - [19/Nov/2026:10:00:01 +0000] "GET /index.html HTTP/1.1" 200 1024 "-" "Mozilla/5.0"
192.168.1.77 - - [19/Nov/2026:10:01:10 +0000] "GET /admin HTTP/1.1" 404 152 "-" "ffuf/v2.1"
192.168.1.77 - - [19/Nov/2026:10:01:10 +0000] "GET /.env HTTP/1.1" 404 152 "-" "ffuf/v2.1"
192.168.1.77 - - [19/Nov/2026:10:01:11 +0000] "GET /backup.zip HTTP/1.1" 404 152 "-" "ffuf/v2.1"
10.0.0.8 - - [19/Nov/2026:10:02:00 +0000] "GET /about.html HTTP/1.1" 200 2048 "-" "Mozilla/5.0"
192.168.1.88 - - [19/Nov/2026:10:03:05 +0000] "GET /item?id=1%20UNION%20SELECT%201,2,3 HTTP/1.1" 500 512 "-" "sqlmap/1.8"
EOF
```

### Step 2: Hunt for Fuzzers (`404` Spikes) & Automated Scanners
```bash
# 1. Rank IPs generating 404 errors
awk '$9 == 404 {print $1}' /tmp/log_lab/access.log | sort | uniq -c | sort -nr

# 2. Hunt for SQLi / scanner signatures in URIs and User-Agents
grep -iE "UNION|SELECT|sqlmap|ffuf|nikto" /tmp/log_lab/access.log

# Clean up
rm -rf /tmp/log_lab
```
*Teaching Moment:* Notice how `$9` in standard Combined Log Format is the HTTP status code (`404`, `500`, `200`) and `$1` is the client IP—`awk` lets you slice logs in seconds without even needing a SIEM!

---

## Practice & Next Steps
- **Labs:** TryHackMe *Splunk: Basics*, *Windows Event Logs*, *Sysmon*, [CyberDefenders](https://cyberdefenders.org/)
- **Next Topic:** [System Hardening & Defense](system_hardening_and_defense.md)
