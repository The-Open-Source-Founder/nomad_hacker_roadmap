# YARA & Volatility 3 — Malware Signatures & Memory Forensics

> **TL;DR:** **YARA** acts as "pattern-matching grep for malware"—identifying malicious files or memory regions using custom rules. **Volatility 3** analyzes raw RAM dumps (`.raw`, `.vmem`, `.dmp`) to uncover hidden processes, network sockets, and in-memory code injection.

---

## 1. YARA — Writing & Running Malware Detection Rules

### Installation
```bash
sudo apt install -y yara
```

### Anatomy of a YARA Rule (`suspicious_ps.yar`)
```yara
rule Suspicious_PowerShell_Downloader {
    meta:
        author = "SOC Team"
        description = "Detects obfuscated PowerShell downloaders"
        severity = "high"

    strings:
        $s1 = "powershell" nocase
        $s2 = "-EncodedCommand" nocase
        $s3 = "Net.WebClient" ascii wide
        $hex_mz = { 4D 5A } // PE executable magic bytes (MZ)

    condition:
        ($s1 and ($s2 or $s3)) or ($hex_mz at 0 and $s3)
}
```

### YARA CLI Cheatsheet
| Command | Purpose |
|---------|---------|
| `yara -r rules.yar /path/to/samples/` | Recursively scan a directory with `rules.yar` |
| `yara -s rules.yar suspect.exe` | Print the **exact matching strings & offsets** (`-s`) |
| `yara -m rules.yar suspect.exe` | Print rule metadata (`-m`) on match |
| `yara rules.yar <PID>` | Scan a live running process's memory |

---

## 2. Volatility 3 — Memory Forensics Workflow

### Installation
```bash
pip3 install volatility3
# Or clone from GitHub: https://github.com/volatilityfoundation/volatility3
```

### Essential Volatility 3 Plugins (Windows Memory Dumps)
*(Volatility 3 auto-detects OS symbols—no `--profile` flag needed like Volatility 2!)*

| Plugin | Purpose |
|--------|---------|
| `windows.info` | Verify OS build, architecture, and capture timestamp |
| `windows.pslist` | List active processes (walks standard linked list; rootkits can unlink here) |
| `windows.pstree` | Show parent-child process tree (spot `winword.exe -> cmd.exe`) |
| `windows.psscan` | Carve memory pools for **hidden or terminated** processes |
| `windows.cmdline` | Print full command-line arguments for every process |
| `windows.netscan` | List active/closed TCP & UDP connections and listening ports |
| `windows.malfind` | Detect **injected code / RWX memory regions** (e.g., Meterpreter/Cobalt Strike) |
| `windows.dlllist --pid <PID>` | List DLLs loaded by a suspicious process |
| `windows.dumpfiles --pid <PID>` | Extract mapped files/executables from RAM for static analysis |

### Step-by-Step Volatility 3 Triage Example
```bash
# 1. Check OS info & process tree
python3 vol.py -f memdump.raw windows.info
python3 vol.py -f memdump.raw windows.pstree

# 2. Check command lines & network connections
python3 vol.py -f memdump.raw windows.cmdline
python3 vol.py -f memdump.raw windows.netscan

# 3. Hunt for process injection
python3 vol.py -f memdump.raw windows.malfind
```

---

## Common Pitfalls & Defensive Notes
- **Overbroad YARA Rules:** A rule matching only `"http://"` or `"cmd.exe"` will trigger thousands of False Positives on legitimate software. Combine multiple specific strings, `filesize < 5MB`, or PE header checks (`uint16(0) == 0x5A4D`).
- **Offline Symbol Cache:** Volatility 3 downloads Windows PDB symbol tables from Microsoft on first run; if analyzing in an air-gapped VM, pre-populate the `volatility3/symbols/` cache.
