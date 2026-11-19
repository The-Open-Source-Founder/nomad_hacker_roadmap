# Sysmon & Windows Event Logs — Endpoint Telemetry

> **TL;DR:** Default Windows Event Logs miss critical attacker activity (like full command lines, DLL loads, and process-initiated network connections). **Sysmon (System Monitor)** fills that gap with rich telemetry logged to `Microsoft-Windows-Sysmon/Operational`.

---

## Installation & Configuration (Windows)
Download Sysmon from **Microsoft Sysinternals** and pair it with a community detection config (e.g., **SwiftOnSecurity** or **Olaf Hartong's sysmon-modular**):
```powershell
# Install Sysmon with a tuned XML configuration
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml

# Update configuration later without reinstalling
.\Sysmon64.exe -c sysmonconfig-export.xml
```

---

## High-Value Event IDs Reference

### Sysmon Event IDs
| Event ID | Name | What It Captures & Why It Matters |
|----------|------|-----------------------------------|
| **1** | Process Creation | Full `CommandLine`, `ParentImage`, `User`, and `SHA256` hash |
| **3** | Network Connection | Process-to-IP/port mapping (spots `powershell.exe` or `rundll32.exe` C2 connections) |
| **7** | Image Loaded | DLLs loaded by processes (detects reflective/unsigned DLL injection) |
| **8** | CreateRemoteThread | One process injecting code into another (classic Cobalt Strike / Meterpreter behavior) |
| **10** | ProcessAccess | Process reading another process's memory (e.g., Mimikatz accessing `lsass.exe`) |
| **11** | FileCreate | New files dropped to disk (e.g., payloads in `C:\Users\Public\` or `Temp\`) |
| **12 / 13 / 14** | Registry Events | Persistence keys created/modified (e.g., `CurrentVersion\Run`) |
| **22** | DNS Query | Which process queried which domain |

### Windows Security Event IDs
| Event ID | Meaning |
|----------|---------|
| **4624 / 4625** | Successful logon / Failed logon (check `LogonType`: `3` = Network/SMB, `10` = RDP) |
| **4688** | Process Creation (enable *Include command line in process creation events* via GPO) |
| **4720 / 4732** | User account created / User added to local group (e.g., Administrators) |
| **7045** | New Windows Service installed (PsExec, lateral movement, persistence) |
| **1102** | Security audit log cleared |

---

## Step-by-Step Practical Workflows (PowerShell `Get-WinEvent`)

### 1. Query Recent Failed Logons (`4625`)
```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} -MaxEvents 15 |
  Select-Object TimeCreated, Message | Format-List
```

### 2. Inspect Sysmon Process Creations (`Event ID 1`)
```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} -MaxEvents 20 |
  ForEach-Object {
    $xml = [xml]$_.ToXml()
    [PSCustomObject]@{
      Time        = $_.TimeCreated
      User        = ($xml.Event.EventData.Data | Where-Object Name -eq 'User').'#text'
      ParentImage = ($xml.Event.EventData.Data | Where-Object Name -eq 'ParentImage').'#text'
      Image       = ($xml.Event.EventData.Data | Where-Object Name -eq 'Image').'#text'
      CommandLine = ($xml.Event.EventData.Data | Where-Object Name -eq 'CommandLine').'#text'
    }
  } | Format-Table -AutoSize
```

### 3. Export Logs for Offline Analysis (`wevtutil`)
```cmd
wevtutil epl Security C:\Temp\security.evtx
wevtutil epl "Microsoft-Windows-Sysmon/Operational" C:\Temp\sysmon.evtx
```

---

## Common Pitfalls & Defensive Notes
- **Installing Sysmon Without a Config:** Running `Sysmon64.exe -i` without an XML filter logs *everything*, flooding the SIEM and overwriting logs within minutes. Always use a curated config.
- **Event Log Tampering:** Forward logs in real time to a SIEM (Winlogbeat / Splunk Universal Forwarder / Wazuh Agent) so attackers can't hide tracks with `wevtutil cl Security` (`Event ID 1102`).
