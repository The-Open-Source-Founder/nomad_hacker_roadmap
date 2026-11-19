# Splunk & SIEM Queries (SPL) — Log Hunting & Detection

> **TL;DR:** **Splunk SPL (Search Processing Language)** pipes search results from left to right (`search | filter | transform | aggregate`). Master `index`, `sourcetype`, `stats`, `eval`, `rex`, `bin`, and `table` to hunt threats and build alerts.

---

## Core SPL Pipeline Anatomy

```text
index=windows sourcetype="WinEventLog:Security" EventCode=4625
| bin _time span=5m
| stats count by _time, Source_Network_Address, Account_Name
| where count >= 10
| sort - count
```
*Think of `|` (pipe) just like Linux Bash pipes: each command processes the output of the previous command.*

---

## Essential SPL Commands Reference

| Command | Purpose & Example |
|---------|-------------------|
| `index=` / `sourcetype=` | Always specify these first to keep searches fast! |
| `stats` | Aggregate data (`stats count by src_ip`, `stats dc(user) by src_ip` for distinct count) |
| `table` | Display only specific columns (`\| table _time, host, user, CommandLine`) |
| `eval` | Create or transform fields (`\| eval mb=round(bytes/1048576, 2)`) |
| `where` | Filter using expressions (`\| where count > 10 AND src_ip!="10.0.0.1"`) |
| `rex` | Extract new fields on-the-fly using Regex |
| `bin` (or `bucket`) | Group timestamps into time windows (`\| bin _time span=15m`) |
| `sort` / `head` | Order results (`\| sort - count \| head 20`) |
| `dedup` | Remove duplicate events by field (`\| dedup src_ip, dest_port`) |

---

## Step-by-Step Threat Hunting Query Templates

### 1. Detect Password Spraying (1 IP Trying Many Usernames)
```spl
index=windows EventCode=4625
| stats dc(Account_Name) as unique_users, count as total_failures by Source_Network_Address
| where unique_users >= 10
| sort - unique_users
```

### 2. Detect Encoded / Suspicious PowerShell Execution
```spl
index=sysmon EventCode=1 Image="*\\powershell.exe"
  (CommandLine="*-enc*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*DownloadString*" OR CommandLine="*IEX*")
| table _time, host, User, ParentImage, CommandLine
```

### 3. Detect Web Directory Fuzzing (`404` Spikes)
```spl
index=web sourcetype=nginx:plus:access status=404
| bin _time span=1m
| stats count by _time, src_ip
| where count > 50
| sort - count
```

### 4. Detect Data Exfiltration (Unusual Outbound Byte Volume)
```spl
index=firewall action=allowed direction=outbound
| stats sum(bytes_out) as total_bytes by src_ip, dest_ip
| eval total_mb = round(total_bytes / 1048576, 2)
| where total_mb > 500
| sort - total_mb
```

---

## Common Pitfalls & Defensive Notes
- **Leading Wildcards (`*evil*`):** Putting `*` at the start of a search term forces Splunk to scan raw text instead of using its index, making queries 10x–100x slower. Put specific `index=`, `sourcetype=`, and `EventCode=` filters first.
- **Alert Fatigue:** Always test a detection query over 7–30 days of historical data and exclude known benign IT automation (e.g., vulnerability scanners) before turning it into a SOC alert.
