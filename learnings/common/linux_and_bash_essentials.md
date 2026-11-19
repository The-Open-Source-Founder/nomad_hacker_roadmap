# Linux & Bash Essentials

> **TL;DR:** Most servers, containers, and security tools run on Linux. Learn how permissions (`rwx`, SUID), key directories (`/etc`, `/var/log`), text-processing pipes (`grep`, `awk`, `sed`, `jq`), and service management (`systemctl`) work.

---

## Core Concepts & Mental Model

### Key Linux Directories
| Path | Purpose | Security Relevance |
|------|---------|-------------------|
| `/etc` | System configuration files | `/etc/passwd`, `/etc/shadow` (hashes), `/etc/sudoers`, `/etc/ssh/sshd_config` |
| `/var/log` | System & application logs | `auth.log` / `secure`, `syslog`, `apache2/`, `nginx/` |
| `/home/<user>` | User home directories | `.ssh/id_rsa`, `.bash_history` |
| `/tmp` & `/dev/shm` | World-writable temp directories | Common staging area for scripts/payloads |
| `/proc` | Virtual filesystem of running processes | Inspect process cmdlines (`/proc/<pid>/cmdline`) and environment |
| `/usr/bin` | User binaries | Check for misconfigured SUID binaries |

### Linux Permissions (`chmod` Numeric & Symbolic)
Each file has permissions for **Owner (`u`)**, **Group (`g`)**, and **Others (`o`)**:
- `r` (Read) = `4`
- `w` (Write) = `2`
- `x` (Execute) = `1`
- Example: `chmod 750 script.sh` → Owner `rwx` (7), Group `r-x` (5), Others `---` (0).
- **Special Bits:**
  - **SUID (`4xxx`, shown as `s` in owner `x`):** Runs the binary with the **file owner's** privileges (often `root`).
  - **SGID (`2xxx`):** Runs with the file group's privileges.

---

## Quick-Reference Command Cheatsheet

### Navigation, Files & Search
| Command | Purpose & Example |
|---------|-------------------|
| `ls -la` | List all files (including hidden `.` files) with permissions and sizes |
| `find / -name "*.conf" 2>/dev/null` | Search filesystem for `.conf` files, hiding permission errors |
| `find / -perm -4000 -type f 2>/dev/null` | Find all SUID binaries on the system |
| `file unknown_bin` | Identify actual file type regardless of extension |
| `strings -n 6 binary` | Extract printable strings (length ≥ 6) from a binary |

### Text Processing & Log Triage
| Command | Purpose & Example |
|---------|-------------------|
| `grep -rn "Failed password" /var/log/` | Recursively search logs with line numbers |
| `awk '{print $1}' access.log \| sort \| uniq -c \| sort -nr \| head` | Count top 10 IP addresses in a web access log |
| `cut -d':' -f1 /etc/passwd` | Extract all usernames from `/etc/passwd` |
| `sed -i 's/old/new/g' file.txt` | Replace text in-place |
| `jq '.[] \| .name' data.json` | Parse and filter JSON output cleanly |

### Users, Processes & Services
| Command | Purpose & Example |
|---------|-------------------|
| `id` / `whoami` | Print current user, UID, GID, and group memberships |
| `sudo -l` | List commands the current user is allowed to run via `sudo` |
| `ps aux` | Show all running processes (`ps aux \| grep python`) |
| `systemctl status ssh` | Check status of a systemd service (`start`, `stop`, `enable`) |
| `crontab -l` | View current user's scheduled cron jobs (also check `/etc/crontab`) |

### Essential Bash Scripting Pattern
```bash
#!/usr/bin/env bash
set -euo pipefail

# Loop over a list of targets from a file
while IFS= read -r host; do
  ping -c 1 -W 1 "$host" &>/dev/null && echo "[UP] $host" || echo "[DOWN] $host"
done < targets.txt
```

---

## Common Pitfalls & Defensive Notes
- **Never ignore `2>/dev/null` blindly when troubleshooting:** Silencing stderr hides real errors; use it intentionally during broad `find` scans.
- **Line Endings (`CRLF` vs `LF`):** Scripts edited on Windows may fail on Linux with `/bin/bash^M: bad interpreter`. Fix with `dos2unix script.sh`.
- **Defensive Tip:** Audit `/etc/sudoers` (`visudo`), monitor `/var/log/auth.log` for `sudo` abuse and failed SSH logins, and mount `/tmp` with `noexec,nosuid` on hardened servers.

---

## Hands-On Lab: Permissions, SUID Hunting & Log Parsing in Bash

**Goal:** Practice creating/modifying permissions, hunting for SUID binaries, and extracting top attacking IPs from a sample log using pipes.

### Step 1: Experiment with Permissions Safely in `/tmp`
```bash
mkdir -p /tmp/linux_lab && cd /tmp/linux_lab
echo '#!/bin/bash' > test.sh
echo 'echo "Executed!"' >> test.sh

# Try running it (fails with Permission Denied)
./test.sh

# Grant owner execute permission (rwx for owner, r-x for group, none for others)
chmod 750 test.sh
ls -la test.sh
./test.sh
```

### Step 2: Hunt for SUID Binaries on Kali
```bash
find / -perm -4000 -type f 2>/dev/null
```
*Teaching Moment:* `-perm -4000` finds files with the SUID bit set, `-type f` restricts to regular files, and `2>/dev/null` hides "Permission denied" noise. Compare your output against [GTFOBins](https://gtfobins.github.io/) during CTFs.

### Step 3: Parse a Simulated Auth Log with `grep`, `awk`, `sort`, `uniq`
```bash
cat << 'EOF' > /tmp/linux_lab/sample_auth.log
Failed password for invalid user admin from 192.168.1.50 port 44122 ssh2
Failed password for root from 10.0.0.99 port 51200 ssh2
Failed password for invalid user test from 192.168.1.50 port 44125 ssh2
Accepted publickey for kali from 10.0.0.5 port 39812 ssh2
Failed password for root from 192.168.1.50 port 44130 ssh2
EOF

# Extract and rank the top attacking IPs
grep "Failed password" /tmp/linux_lab/sample_auth.log | awk '{for(i=1;i<=NF;i++) if($i=="from") print $(i+1)}' | sort | uniq -c | sort -nr
```
*Expected Output:* `3 192.168.1.50` followed by `1 10.0.0.99`.

```bash
# Clean up lab directory
rm -rf /tmp/linux_lab
```

---

## Practice & Next Steps
- **Labs:** [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/), TryHackMe *Linux Fundamentals 1–3*
- **Next Topic:** [Windows & Active Directory Basics](windows_and_active_directory_basics.md)
