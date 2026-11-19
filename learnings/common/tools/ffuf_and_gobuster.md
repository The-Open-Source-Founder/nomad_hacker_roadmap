# ffuf & Gobuster — Web Content, Directory & VHost Fuzzing

> **TL;DR:** `ffuf` (Fuzz Faster U Fool) and `gobuster` brute-force hidden directories, files, parameters, and Virtual Hosts (VHosts) using wordlists. `ffuf` is preferred for its speed and powerful response-size/word filtering.

---

## Installation & Wordlists
```bash
sudo apt install -y ffuf gobuster seclists
```
Standard wordlists live in `/usr/share/seclists/Discovery/Web-Content/` and `/usr/share/wordlists/dirb/`.

---

## Essential `ffuf` Flags Reference

| Flag | Description |
|------|-------------|
| `-u <url>` | Target URL containing the `FUZZ` keyword (e.g., `http://target.com/FUZZ`) |
| `-w <wordlist>` | Path to wordlist (`-w list.txt:FUZZ` for custom keyword) |
| `-H "Header: val"` | Add custom HTTP header (used for VHost fuzzing or auth cookies) |
| `-mc <codes>` | **Match** HTTP status codes (default: `200,204,301,302,307,401,403`) |
| `-fc <codes>` | **Filter out** specific status codes (e.g., `-fc 404,429`) |
| `-fs <bytes>` | **Filter out** responses of an exact byte size (critical for soft-404s & VHosts) |
| `-fw <words>` | **Filter out** responses with an exact word count |
| `-e .php,.txt,.bak` | Append file extensions to every word in the list |
| `-t <num>` | Concurrent threads (default `40`; lower to `10` if rate-limited) |
| `-recursion` | Automatically fuzz inside discovered directories |

---

## Step-by-Step Practical Workflows

### 1. Directory & File Discovery (`ffuf`)
```bash
ffuf -u http://target.local/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -mc 200,204,301,302,307,401,403
```

### 2. Fuzzing Specific File Extensions
```bash
ffuf -u http://target.local/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -e .php,.html,.txt,.bak,.zip
```

### 3. Virtual Host (VHost) Fuzzing
First, check the baseline response size of a nonexistent VHost (e.g., `random123.target.local`), then filter that size with `-fs`:
```bash
ffuf -u http://10.10.10.5 \
  -H "Host: FUZZ.target.local" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fs 1542
```

### 4. GET Parameter Fuzzing
Discover hidden query parameters (e.g., `?debug=1` or `?file=`):
```bash
ffuf -u "http://target.local/index.php?FUZZ=1" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -fs 3200
```

### 5. Gobuster Equivalents
| Mode | Command |
|------|---------|
| **Directory** | `gobuster dir -u http://target.local -w /usr/share/wordlists/dirb/common.txt -x php,txt -t 30` |
| **DNS Subdomain** | `gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` |
| **VHost** | `gobuster vhost -u http://target.local -w subdomains.txt --append-domain` |

---

## Common Pitfalls & Defensive Notes
- **Soft-404 Floods:** If every word returns `200 OK`, stop immediately and filter by baseline size (`-fs`) or word count (`-fw`).
- **Rate Limiting (`429`) / WAF Bans:** High thread counts (`-t 100`) will get your IP banned. Use `-rate 10` (requests/sec) or `-p 0.1` (delay) for stealthier scans.
- **Defensive Tip:** Configure WAF/rate-limiting rules to alert on high-frequency `404` bursts from a single IP, and disable directory listing (`autoindex off` in Nginx).
