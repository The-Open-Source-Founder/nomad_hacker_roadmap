# Nmap — Network Mapper & Service Enumeration

> **TL;DR:** `nmap` discovers live hosts, open TCP/UDP ports, exact service versions (`-sV`), OS fingerprints (`-O`), and runs automated **NSE (Nmap Scripting Engine)** checks (`-sC`, `--script`).

---

## Installation & Availability
Pre-installed on Kali Linux. Cross-platform for Linux, macOS, and Windows.
```bash
sudo apt install -y nmap
```

---

## Essential Flags Reference

| Flag | Category | Description |
|------|----------|-------------|
| `-sn` | Host Discovery | Ping sweep only (no port scan) |
| `-Pn` | Host Discovery | Treat all hosts as online (skip ping; essential when ICMP is blocked) |
| `-sS` | Scan Type | **TCP SYN (Stealth) scan** — half-open, fast, requires `sudo` |
| `-sT` | Scan Type | **TCP Connect scan** — completes 3-way handshake, no `sudo` needed |
| `-sU` | Scan Type | **UDP scan** (slower; pair with `--top-ports 50`) |
| `-p-` | Ports | Scan all 65,535 TCP ports (`-p 22,80,443` for specific ports) |
| `-sV` | Enumeration | Probe open ports to determine service/version info |
| `-sC` | Enumeration | Run default safe NSE scripts (equivalent to `--script=default`) |
| `-O` | Enumeration | OS detection (requires `sudo`) |
| `-A` | Aggressive | Combines `-sV -sC -O --traceroute` (very noisy!) |
| `-T0`..`-T5` | Timing | Speed template (`-T4` is standard for fast labs; `-T1`/`-T2` for stealth) |
| `--min-rate <n>` | Timing | Send at least `<n>` packets per second (e.g., `--min-rate 2000`) |
| `-oA <prefix>` | Output | Save results in all 3 formats (`.nmap`, `.gnmap`, `.xml`) |

---

## Step-by-Step Practical Workflows

### 1. The Standard 2-Step Pentest Scan
**Step 1:** Fast full-port discovery to avoid missing high ports:
```bash
sudo nmap -sS -p- --min-rate 2000 -Pn -oA all_ports 10.10.10.5
```
**Step 2:** Targeted deep scan (`-sC -sV`) only on the open ports found in Step 1:
```bash
sudo nmap -sC -sV -p 22,80,445,8080 -oA detailed_scan 10.10.10.5
```

### 2. Ping Sweep a Subnet
```bash
nmap -sn 10.10.10.0/24 -oG live_hosts.gnmap
grep "Up" live_hosts.gnmap | awk '{print $2}' > ips.txt
```

### 3. Top UDP Services Scan
```bash
sudo nmap -sU --top-ports 50 --open 10.10.10.5
```

### 4. Using NSE (Nmap Scripting Engine) Scripts
NSE scripts live in `/usr/share/nmap/scripts/`. Search them with `ls /usr/share/nmap/scripts/ | grep smb`:
```bash
# Check for SMB vulnerabilities (e.g., EternalBlue MS17-010)
nmap -p 445 --script smb-vuln* 10.10.10.5

# Brute-force DNS subdomains
nmap --script dns-brute example.com

# Enumerate HTTP titles & methods
nmap -p 80,443 --script http-title,http-methods 10.10.10.5
```

---

## Common Pitfalls & Defensive Notes
- **Running `-A` on All 65,535 Ports:** `nmap -A -p-` can take hours and crash fragile services. Always separate **port discovery** (`-p-`) from **service enumeration** (`-sC -sV`).
- **Defensive Tip:** Nmap scans generate rapid `SYN` bursts across sequential/random ports, triggering IDS/IPS alerts (Suricata/Snort) and firewall rate-limits.
