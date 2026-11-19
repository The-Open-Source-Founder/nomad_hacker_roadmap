# Web & HTTP Fundamentals

> **TL;DR:** Web applications are the #1 external attack surface. Understand HTTP requests/responses, status codes, cookies/JWTs, TLS/HTTPS, and Same-Origin Policy (SOP)/CORS, and practice inspecting traffic with `curl` and Burp Suite.

---

## Core Concepts & Mental Model

### HTTP Request & Response Anatomy
```http
POST /api/login HTTP/1.1
Host: target.local
Content-Type: application/json
Cookie: session=abc123

{"username":"admin","password":"secret"}
```

| Status Code Range | Meaning | Common Examples |
|-------------------|---------|-----------------|
| `2xx` | Success | `200 OK`, `201 Created`, `204 No Content` |
| `3xx` | Redirection | `301 Moved Permanently`, `302 Found` |
| `4xx` | Client Error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found` |
| `5xx` | Server Error | `500 Internal Server Error`, `502 Bad Gateway` |

### Sessions: Cookies vs. JWT (JSON Web Tokens)
- **Session Cookies:** Server stores session state; browser sends `session_id` automatically. Must have `HttpOnly` (blocks JS access/XSS theft), `Secure` (HTTPS only), and `SameSite=Lax/Strict` (mitigates CSRF).
- **JWT (`header.payload.signature`):** Stateless, Base64URL-encoded token. Anyone can **decode** the payload; only the server should be able to **verify/sign** the signature.

### Same-Origin Policy (SOP) & CORS
- **Origin** = `Scheme + Hostname + Port` (e.g., `https://app.example.com:443`).
- **SOP** prevents `evil.com` JavaScript from reading responses from `bank.com`.
- **CORS (`Access-Control-Allow-Origin`)** relaxes SOP intentionally; misconfigured wildcard/reflected origins (`ACAO: *` with `Allow-Credentials: true`) allow cross-origin data theft.

---

## Quick-Reference Command Cheatsheet (`curl` & TLS)

| Command | Purpose |
|---------|---------|
| `curl -I https://example.com` | Fetch **HTTP response headers only** (`HEAD` request) |
| `curl -i -s https://example.com` | Show headers + body (`-i`) silently (`-s`) |
| `curl -L https://example.com` | Follow `301`/`302` redirects automatically |
| `curl -X POST -H "Content-Type: application/json" -d '{"user":"test"}' https://example.com/api` | Send a JSON `POST` request |
| `curl -H "Host: dev.example.com" http://10.10.10.5` | Test **Virtual Host (VHost)** routing directly against an IP |
| `curl -x http://127.0.0.1:8080 -k https://example.com` | Route `curl` traffic through Burp Suite proxy (`-k` ignores self-signed cert) |
| `openssl s_client -connect example.com:443 -brief` | Inspect TLS version, cipher suite, and certificate chain |

---

## Common Pitfalls & Defensive Notes
- **Assuming `403 Forbidden` is Unbypassable:** Sometimes `403` is enforced only at a reverse proxy/WAF path rule (e.g., `/admin` vs `/admin/` or `X-Original-URL`). Always enforce authorization in backend application logic.
- **Defensive Tip:** Set `Strict-Transport-Security` (HSTS), `Content-Security-Policy` (CSP), `X-Content-Type-Options: nosniff`, and `HttpOnly; Secure; SameSite=Lax` on all session cookies.

---

## Hands-On Lab: Inspecting HTTP Headers, Cookies & TLS from Kali

**Goal:** Spin up a quick local HTTP server, inspect raw requests/responses with `curl`, decode a JWT, and inspect a live TLS certificate.

### Step 1: Start a Local Test Server & Inspect Headers
Open two Kali terminal panes:
```bash
# Terminal 1: Start a local Python HTTP server on port 8000
python3 -m http.server 8000
```
```bash
# Terminal 2: Fetch headers only (-I) and send a custom header (-H)
curl -I http://127.0.0.1:8000/
curl -v -H "X-Custom-Test: KaliLab" http://127.0.0.1:8000/
```
*Teaching Moment:* Look at Terminal 1 to see how the server logs your `GET / HTTP/1.1` and `200` status code, and look at Terminal 2 (`-v`) to see `>` (outgoing request headers) and `<` (incoming response headers). Stop Terminal 1 with `Ctrl+C` when done.

### Step 2: Decode a Sample JWT Token in Bash
```bash
# Sample JWT (Header.Payload.Signature)
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMDQiLCJ1c2VyIjoiYWxpY2UiLCJyb2xlIjoidXNlciJ9.signature"

# Split on '.' and Base64-decode the payload (field 2)
echo "$TOKEN" | cut -d'.' -f2 | base64 -d 2>/dev/null; echo
```
*Expected Output:* `{"sub":"104","user":"alice","role":"user"}` — proving anyone can read a JWT payload without knowing the secret!

### Step 3: Inspect a Live TLS Certificate
```bash
echo | openssl s_client -connect example.com:443 -servername example.com -brief
```

---

## Practice & Next Steps
- **Labs:** PortSwigger Web Security Academy (*HTTP Host header*, *CORS*, *JWT*), TryHackMe *HTTP in Detail*
- **Next Topic:** [Cryptography Basics](cryptography_basics.md)
