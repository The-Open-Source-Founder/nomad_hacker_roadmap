# Offensive Track Tools

Detailed, hands-on reference guides for core penetration testing and red-team utilities.

| Tool | Phase | What You'll Learn | Guide |
|------|-------|-------------------|-------|
| **OSINT & Social Eng Tools** | Recon & Awareness | `theHarvester`, `sherlock`, `exiftool` metadata extraction, `setoolkit` | [setoolkit_and_osint_tools.md](setoolkit_and_osint_tools.md) |
| **Nmap** | Scanning & Enum | Host discovery, SYN/TCP/UDP scans, NSE scripts, timing, output formats | [nmap.md](nmap.md) |
| **Nuclei, Nikto & WPScan** | Vulnerability Scanning | YAML CVE/misconfig templates, web server checks, WordPress enum | [nuclei_and_nikto.md](nuclei_and_nikto.md) |
| **Burp Suite** | Web App Hacking | Proxy/CA setup, Scope, Intercept, Repeater, Intruder, Decoder | [burp_suite.md](burp_suite.md) |
| **SQLMap** | Web Exploitation | GET/POST/`-r` request testing, DB dumping, tamper scripts, OS shells | [sqlmap.md](sqlmap.md) |
| **Metasploit & msfvenom** | Exploitation | `msfconsole` workflow, module options, sessions, standalone payload generation | [metasploit.md](metasploit.md) |
| **Chisel & Ligolo-ng** | Pivoting / Tunneling | Virtual TUN routing (`ligolo-ng`), SOCKS5 tunneling (`chisel` + `proxychains4`) | [chisel_and_ligolo.md](chisel_and_ligolo.md) |
| **Hydra & NetExec (`nxc`)** | Password Attacks | Online brute-forcing, SMB/WinRM enumeration, safe password spraying | [hydra_and_netexec.md](hydra_and_netexec.md) |
| **Responder & Impacket** | Internal / AD | LLMNR poisoning, `impacket-smbserver`, Kerberoasting, `secretsdump`, `wmiexec` | [responder_and_impacket.md](responder_and_impacket.md) |
| **BloodHound & AD Enum** | Active Directory | `bloodhound-python`, Neo4j attack path graphing, `kerbrute`, `ldapsearch` | [bloodhound_and_ad_enum.md](bloodhound_and_ad_enum.md) |
| **Hashcat & John** | Offline Cracking | GPU hash modes (`-m`), rule mutations, `unshadow`, potfile management | [hashcat_and_john.md](hashcat_and_john.md) |
| **LinPEAS, WinPEAS & Mimikatz** | Post-Exploit & PrivEsc | Automated PrivEsc enum (`/usr/share/peass/`), LSASS/SAM dumping, `pypykatz` | [privesc_and_cred_tools.md](privesc_and_cred_tools.md) |
| **Aircrack-ng & Wi-Fi Tools** | Wireless Attacks | Monitor mode (`airmon-ng`), `airodump-ng`, `aireplay-ng`, `hcxdumptool`, `wifite`, `reaver` | [aircrack_and_wifi_tools.md](aircrack_and_wifi_tools.md) |

[← Back to Master Roadmap](../../../README.md)
