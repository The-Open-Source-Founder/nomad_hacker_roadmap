# Nuclei, Nikto & WPScan — Automated Web Vulnerability Scanners

> **TL;DR:** **Nuclei** is the modern, fast, YAML-template-based vulnerability scanner for CVEs, exposed panels, and misconfigurations. **Nikto** is Kali's classic web server misconfiguration scanner, and **WPScan** is the specialized scanner for WordPress sites.

---

## Installation & Availability in Kali Linux
```bash
sudo apt install -y nuclei nikto wpscan
# Update Nuclei templates before scanning:
nuclei -update-templates
```

---

## 1. Nuclei — Template-Based Vulnerability Scanning

| Flag | Description |
|------|-------------|
| `-u <url>` / `-l <file>` | Scan a single URL (`-u`) or a list of URLs (`-l`) |
| `-severity <levels>` | Filter templates (`info,low,medium,high,critical`) |
| `-tags <tags>` | Run specific categories (e.g., `cve,misconfig,exposure,panel`) |
| `-rl <rate>` | Rate limit requests per second (e.g., `-rl 50` to avoid crashing targets) |
| `-o <file>` | Save findings to a text file |

### Practical Nuclei Workflows
```bash
# Scan a target for medium, high, and critical vulnerabilities
nuclei -u https://target.local -severity medium,high,critical -o nuclei_results.txt

# Check only for exposed admin panels, .git, .env, and misconfigurations
nuclei -u https://target.local -tags exposure,misconfig,panel

# Pipeline: subfinder -> httpx -> nuclei (classic Kali bug-bounty pipeline)
subfinder -d target.com -silent | httpx -silent | nuclei -severity high,critical
```

---

## 2. Nikto — Classic Web Server Misconfiguration Scanner

| Flag | Description |
|------|-------------|
| `-h <url_or_ip>` | Target host/URL |
| `-p <port>` | Target port (e.g., `-p 8080` or `-p 80,443`) |
| `-ssl` | Force SSL/HTTPS mode |
| `-o <file> -Format txt` | Save output to file (`txt`, `html`, `csv`) |

```bash
nikto -h http://10.10.10.5 -o nikto_scan.txt
```

---

## 3. WPScan — WordPress Security Scanner

```bash
# Enumerate WordPress users, vulnerable plugins, and themes
wpscan --url http://target.local --enumerate u,vp,vt

# Brute-force WordPress login (with rockyou.txt)
wpscan --url http://target.local -U admin -P /usr/share/wordlists/rockyou.txt
```
*(Tip: Register a free WPScan API token to pull live CVE data for detected plugins with `--api-token <TOKEN>`.)*

---

## Common Pitfalls & Defensive Notes
- **Extreme Noise:** `nikto` and `nuclei` send thousands of known-malicious paths and payloads—every WAF and SIEM will log them immediately.
- **Defensive Tip:** Keep CMS core/plugins updated, remove `.git`/`.env`/backup files from web roots, and monitor logs for `Nikto` or `Nuclei` User-Agent strings.
