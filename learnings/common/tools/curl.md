# cURL — Command-Line HTTP & Network Client

> **TL;DR:** `curl` transfers data over HTTP/HTTPS (and FTP, SMB, etc.) directly from the terminal. In security, it is essential for inspecting raw headers, testing APIs, spoofing `Host`/`User-Agent` headers, debugging TLS, and routing requests through Burp Suite.

---

## Installation & Availability
Pre-installed on Kali Linux, Ubuntu, macOS, and Windows 10/11 (`curl.exe`).
```bash
sudo apt install -y curl
```

---

## Essential Flags Reference

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-I` | `--head` | Fetch **HTTP headers only** (`HEAD` request) |
| `-i` | `--include` | Include response headers above the response body |
| `-s` | `--silent` | Hide progress meter and error messages |
| `-v` | `--verbose` | Show full TLS handshake, request headers, and response headers |
| `-L` | `--location` | Automatically follow `301`/`302` HTTP redirects |
| `-X` | `--request` | Specify HTTP method (`GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`) |
| `-H` | `--header` | Add or override a custom HTTP header (e.g., `Host:`, `Authorization:`) |
| `-d` | `--data` | Send `POST` body data (form-encoded or JSON) |
| `-b` / `-c` | `--cookie` / `--cookie-jar` | Send cookies (`-b`) or save received `Set-Cookie` headers to a file (`-c`) |
| `-k` | `--insecure` | Skip TLS certificate verification (useful for self-signed lab certs) |
| `-x` | `--proxy` | Route request through an HTTP/SOCKS proxy (e.g., `http://127.0.0.1:8080`) |
| `-o` / `-O` | `--output` | Save response to a custom filename (`-o file`) or remote filename (`-O`) |

---

## Step-by-Step Practical Workflows

### 1. Fingerprint a Web Server & Security Headers
```bash
curl -I -s -L https://example.com
```
Look for `Server:`, `X-Powered-By:`, `Strict-Transport-Security`, `Content-Security-Policy`, and `Set-Cookie` flags (`HttpOnly; Secure; SameSite`).

### 2. Test Virtual Host (VHost) Routing
When an IP hosts multiple websites without public DNS entries:
```bash
curl -i -s -H "Host: admin.internal.local" http://10.10.10.5/
```

### 3. Send Authenticated JSON API Requests
```bash
curl -s -X POST https://target.local/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"Password123!"}'
```
Using a Bearer token from the response:
```bash
curl -s https://target.local/api/v1/users \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." | jq .
```

### 4. Save & Reuse Session Cookies
```bash
# Log in and save session cookie to cookies.txt
curl -s -c cookies.txt -X POST http://target.local/login -d "user=alice&pass=secret"
# Access protected endpoint using saved cookie
curl -s -b cookies.txt http://target.local/dashboard
```

### 5. Route Through Burp Suite Proxy
```bash
curl -x http://127.0.0.1:8080 -k https://target.local/api/test
```

### 6. Check Allowed HTTP Methods (`OPTIONS`)
```bash
curl -i -s -X OPTIONS http://target.local/uploads/
```
Watch for `Allow: GET, POST, PUT, DELETE`—`PUT` or `DELETE` on an upload folder can indicate dangerous misconfigurations.

---

## Common Pitfalls & Defensive Notes
- **URL Encoding:** Special characters (`&`, `?`, spaces) in `-d` or URLs can break shell parsing. Always quote URLs and use `--data-urlencode` when needed.
- **Default User-Agent:** `curl/8.x.x` stands out in web access logs and is often blocked by WAFs. Spoof a browser header with `-A "Mozilla/5.0 ..."` when authorized.
- **Defensive Tip:** Defenders can hunt for automated probing in Nginx/Apache logs by filtering for `curl/` or `python-requests/` User-Agents hitting sensitive paths like `/.env` or `/admin`.
