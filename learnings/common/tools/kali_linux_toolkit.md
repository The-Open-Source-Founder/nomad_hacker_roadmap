# Kali Linux Toolkit, Wordlists & Built-In Resources

> **TL;DR:** Kali Linux ships with far more than scanners—it includes pre-staged wordlists (`/usr/share/wordlists`, `SecLists`), webshells (`/usr/share/webshells`), Windows binaries (`/usr/share/windows-resources`), `searchsploit`, `impacket-*` scripts, `proxychains4`, and `tmux`. Knowing where Kali stores things saves huge amounts of time.

---

## 1. Kali Metapackages (`apt`)

Install curated toolsets without hunting individual packages:
| Metapackage | What It Installs |
|-------------|------------------|
| `kali-linux-default` | Standard Kali desktop tools (`nmap`, `burpsuite`, `wireshark`, `metasploit-framework`, `sqlmap`) |
| `kali-linux-large` | Extended offensive/defensive toolkit + extra wordlists |
| `kali-tools-top10` | Top 10 most common pentesting tools (great for minimal containers/VMs) |
| `kali-tools-web` | Web app testing suite (`ffuf`, `gobuster`, `nikto`, `wpscan`, `burpsuite`) |
| `kali-tools-passwords` | Offline & online password tools (`hashcat`, `john`, `hydra`, `wordlists`) |
| `kali-tools-forensics` | DFIR & triage suite (`volatility3`, `yara`, `binwalk`, `sleuthkit`) |

```bash
# Configure Kali defaults, shell prompts, and metapackages interactively:
kali-tweaks
```

---

## 2. Essential `/usr/share/` Resource Paths in Kali

| Path | What's Inside |
|------|---------------|
| `/usr/share/wordlists/rockyou.txt` | Classic password wordlist (run `sudo gzip -d /usr/share/wordlists/rockyou.txt.gz` once!) |
| `/usr/share/seclists/` | **SecLists** (`Discovery/Web-Content/`, `Discovery/DNS/`, `Passwords/`, `Usernames/`) |
| `/usr/share/webshells/` | Ready-to-use PHP, ASP, JSP, and Perl reverse/web shells |
| `/usr/share/windows-resources/` | Pre-compiled Windows binaries (`nc.exe`, `mimikatz`, `binaries/whoami.exe`, `powersploit`) |
| `/usr/share/nmap/scripts/` | All Nmap NSE scripts (`ls /usr/share/nmap/scripts/ \| grep smb`) |
| `/usr/share/exploitdb/` | Local Exploit-DB archive queried by `searchsploit` |

---

## 3. Essential Kali Workflow Utilities

### `tmux` (Terminal Multiplexer — Split Panes & Persistent Sessions)
| Shortcut / Command | Action |
|--------------------|--------|
| `tmux new -s box1` | Start a named session (e.g., for a target machine) |
| `Ctrl+B` then `%` / `"` | Split pane vertically / horizontally |
| `Ctrl+B` then Arrow Key | Switch between panes |
| `Ctrl+B` then `d` | Detach from session (processes keep running!) |
| `tmux attach -t box1` | Re-attach to session |

### `searchsploit` (Local Exploit-DB Search)
```bash
searchsploit -u                      # Update local Exploit-DB database
searchsploit apache 2.4.49           # Search by software & version
searchsploit -m 50383                # Copy exploit 50383 into current directory
searchsploit -x 50383                # Inspect exploit source code before running
```

### `impacket-*` Suite (Kali's Built-In AD/SMB Commands)
Kali exposes Impacket scripts globally with the `impacket-` prefix:
| Command | Purpose |
|---------|---------|
| `impacket-smbserver share . -smb2support` | Spin up a quick SMB server on Kali to transfer files to/from Windows targets |
| `impacket-psexec corp.local/admin:Pass@10.10.10.10` | Remote SYSTEM shell via SMB (requires admin creds) |
| `impacket-secretsdump corp.local/admin:Pass@10.10.10.10` | Dump SAM/LSA/NTDS hashes remotely |
| `impacket-GetNPUsers corp.local/ -usersfile users.txt -no-pass` | AS-REP Roasting (find AD users without Kerberos pre-auth) |

### `proxychains4` (Pivoting Through SOCKS Proxies)
Edit `/etc/proxychains4.conf` (default: `socks4 127.0.0.1 9050` or `socks5 127.0.0.1 1080`), then prefix any Kali tool:
```bash
proxychains4 -q nmap -sT -Pn 172.16.10.5
proxychains4 -q curl http://172.16.10.5
```
*(Note: Always use `-sT` TCP Connect scan with `proxychains4`; `-sS` SYN scans do not work over SOCKS.)*

### `openvpn` (Connecting to TryHackMe / Hack The Box)
```bash
sudo openvpn ~/Downloads/lab_vpn.ovpn
ip -4 addr show tun0   # Confirm your tun0 IP (use this as LHOST!)
```

---

## Common Pitfalls & Tips
- **Default Credentials:** Change the default `kali:kali` password immediately with `passwd` if your VM is ever on a shared network.
- **Using `eth0` Instead of `tun0` for `LHOST`:** When on a lab VPN, always pass your `tun0` IP to `msfvenom`, `nc`, and reverse shell payloads.
