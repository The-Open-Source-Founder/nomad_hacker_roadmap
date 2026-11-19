# Hashcat & John the Ripper — Offline Password Cracking

> **TL;DR:** Once you dump password hashes (from `/etc/shadow`, SAM, NTDS.dit, Kerberoasting, or web DBs), crack them offline. **Hashcat** leverages GPUs for massive speed; **John the Ripper** excels at quick CPU cracking and formats like ZIP/SSH keys.

---

## Installation & Availability
Pre-installed on Kali Linux.
```bash
sudo apt install -y hashcat john
```

---

## 1. Hashcat — Essential Flags & Attack Modes

| Flag | Description |
|------|-------------|
| `-m <mode>` | Hash type ID (e.g., `0` = MD5, `1000` = NTLM, `1800` = sha512crypt, `3200` = bcrypt, `13100` = Kerberoast) |
| `-a <mode>` | Attack mode: `0` = Dictionary (Wordlist), `3` = Brute-Force/Mask, `6` = Hybrid (Wordlist + Mask) |
| `-r <rule>` | Apply rule file to mutate wordlist (e.g., `/usr/share/hashcat/rules/best64.rule`) |
| `--show` | Display already-cracked passwords from `hashcat.potfile` |
| `-O` | Enable optimized kernels (faster, limits max password length to 31) |

### Hashcat Examples
```bash
# Crack Windows NTLM hashes with rockyou.txt
hashcat -m 1000 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt

# Crack NTLM with rule mutations (capitalization, appended numbers/symbols)
hashcat -m 1000 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Mask attack: 8-char password starting with uppercase, 5 lowercase, 2 digits
hashcat -m 0 md5.txt -a 3 ?u?l?l?l?l?l?d?d

# View cracked results
hashcat -m 1000 ntlm_hashes.txt --show
```

---

## 2. John the Ripper — Essential Workflows

John automatically detects many hash formats and includes `*2john` utilities to extract hashes from protected files.

| Command | Purpose |
|---------|---------|
| `unshadow passwd.txt shadow.txt > unshadowed.txt` | Merge Linux `/etc/passwd` & `/etc/shadow` for cracking |
| `john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt` | Crack hashes using a wordlist |
| `john --format=NT --wordlist=rockyou.txt ntlm.txt` | Force a specific hash format when auto-detection is ambiguous |
| `zip2john secret.zip > zip.hash && john zip.hash` | Extract & crack a password-protected ZIP archive |
| `ssh2john id_rsa > ssh.hash && john --wordlist=rockyou.txt ssh.hash` | Crack an encrypted SSH private key passphrase |
| `john --show hashes.txt` | Display cracked passwords stored in `~/.john/john.pot` |

---

## Common Pitfalls & Defensive Notes
- **Running Hashcat inside a VM Without GPU:** VirtualBox/VMware VMs usually lack GPU passthrough; Hashcat may error on OpenCL/CUDA or run slowly on CPU (`--force` bypasses CPU warnings in labs, or use `john`).
- **Defensive Tip:** Use **Argon2id** or **bcrypt** for application passwords, enforce **14+ character passphrases** for AD/SSH keys, and monitor for Kerberoasting (`Event ID 4769` with RC4 `0x17` encryption).
