# System Hardening & Defense (Defensive)

> **TL;DR:** Hardening shrinks the attack surface *before* an incident happens. Apply **Least Privilege**, disable legacy/insecure protocols (Telnet, SMBv1, LLMNR), enforce key-based SSH + firewalls, keep systems patched, and validate against **CIS Benchmarks**.

---

## Core Concepts & Mental Model

### Core Security Principles
- **Principle of Least Privilege (PoLP):** Every user, service, and process gets only the minimum permissions required to do its job.
- **Defense in Depth:** Layer controls (Network Firewall -> Host Firewall -> MFA -> EDR -> File Permissions) so one failure doesn't mean total compromise.
- **Attack Surface Reduction:** Disable unused services, ports, and features—what isn't running can't be exploited.

---

## Quick-Reference Hardening Cheatsheet

### 1. Linux Server Hardening
| Control | Command / Configuration |
|---------|-------------------------|
| **Host Firewall (UFW)** | `sudo ufw default deny incoming && sudo ufw allow 22/tcp && sudo ufw enable` |
| **Disable SSH Root & Passwords** | In `/etc/ssh/sshd_config`: set `PermitRootLogin no` and `PasswordAuthentication no` (use SSH keys!), then `sudo systemctl reload ssh` |
| **Brute-Force Protection** | `sudo apt install fail2ban && sudo systemctl enable --now fail2ban` |
| **Automatic Security Updates** | `sudo apt install unattended-upgrades && sudo dpkg-reconfigure -plow unattended-upgrades` |
| **Audit Open Ports & SUIDs** | `ss -tulnp` and `find / -perm -4000 -type f 2>/dev/null` |

### 2. Windows & Active Directory Hardening
| Control | Command / Setting |
|---------|-------------------|
| **Enable Windows Firewall** | `Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True` |
| **Disable SMBv1 (EternalBlue)** | `Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol` |
| **Disable LLMNR / NetBIOS** | GPO: *Turn off multicast name resolution = Enabled* (prevents Responder/NTLM relay) |
| **Enable PowerShell Logging** | GPO: *Turn on PowerShell Script Block Logging* (`Event ID 4104`) |
| **Attack Surface Reduction (ASR)** | Use Microsoft Defender ASR rules to block Office apps from spawning child processes (`cmd`/`powershell`) |

---

## Common Pitfalls & Defensive Notes
- **Locking Yourself Out via Firewall/SSH:** Always test SSH key login in a *second* terminal before setting `PasswordAuthentication no`, and always `ufw allow 22/tcp` *before* running `ufw enable`.
- **Hardening Without Monitoring:** Hardening reduces noise, but determined attackers adapt. Pair hardening with centralized logging and regular vulnerability scanning.

---

## Hands-On Lab: Audit Your Kali / Linux System with `lynis` & `ss`

**Goal:** Perform a baseline security audit on your Linux VM using `lynis` (automated CIS/hardening auditor) and inspect listening services and SSH config.

### Step 1: Audit Listening Ports & SSH Configuration
```bash
# 1. List all listening TCP/UDP sockets
sudo ss -tulnp

# 2. Check critical SSH hardening settings (PermitRootLogin & PasswordAuthentication)
grep -E "^(PermitRootLogin|PasswordAuthentication)" /etc/ssh/sshd_config
```

### Step 2: Run a Local Hardening Audit with `lynis`
```bash
sudo apt install -y lynis
sudo lynis audit system --quick
```
*Teaching Moment:* `lynis` scores your system's **Hardening Index** and prints actionable `[WARNING]` and `[SUGGESTION]` items (such as kernel sysctl tuning, file permissions, and disabled firewall rules).

---

## Practice & Next Steps
- **Standards:** [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks), [Lynis](https://cisofy.com/lynis/) (automated Linux security auditing tool)
- **Next Topic:** [Malware & Forensics Basics](malware_and_forensics_basics.md)
