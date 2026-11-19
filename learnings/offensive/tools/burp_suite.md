# Burp Suite — Web Application Security Testing Proxy

> **TL;DR:** Burp Suite sits as a Man-in-the-Middle (MitM) HTTP/HTTPS proxy between your browser and the target web app. Master **Proxy (Intercept & History)**, **Target Scope**, **Repeater**, and **Intruder**—they are the core of manual web pentesting.

---

## Installation & Browser Setup
Burp Suite Community Edition is pre-installed on Kali Linux (`burpsuite`).

### Initial HTTPS Proxy Setup (One-Time)
1. Launch `burpsuite` -> **Temporary project** -> **Use Burp defaults**.
2. Go to **Proxy -> Proxy settings** and confirm it listens on `127.0.0.1:8080`.
3. Configure your browser (or **FoxyProxy** extension) to route HTTP/HTTPS through `127.0.0.1:8080` (or use Burp's built-in **Open Browser** button).
4. **Install Burp CA Certificate** (required for HTTPS):
   - Visit `http://burpsuite` in the proxied browser -> download **CA Certificate** (`cacert.der`).
   - Import it into Firefox/Chrome **Authorities** trust store.

---

## Core Tabs & Feature Breakdown

| Tab | Purpose | When to Use |
|-----|---------|-------------|
| **Proxy -> Intercept** | Pause, inspect, and edit live requests/responses before they leave or render | Modifying hidden form fields or bypassing client-side JS validation |
| **Proxy -> HTTP history** | Complete log of all requests/responses | Reviewing API calls, parameters, headers, and cookies passively |
| **Target -> Site map** | Tree view of discovered hosts, endpoints, and parameters | Set **Add to scope** here to filter out noisy third-party browser traffic |
| **Repeater (`Ctrl+R`)** | Manually edit and resend a single request repeatedly | Testing **IDOR**, **SQLi**, **SSRF**, **XSS**, **JWT**, and auth bypasses |
| **Intruder (`Ctrl+I`)** | Automated parameter fuzzing and brute-forcing | Fuzzing IDs, PINs, or usernames (note: throttled in Community Edition; use `ffuf` for large wordlists) |
| **Decoder** | Encode/decode Base64, URL, Hex, HTML, and hashes | Decoding session cookies, JWT payloads, or URL-encoded payloads |
| **Comparer** | Diff two HTTP responses side-by-side | Spotting subtle differences in blind SQLi or error-based enumeration |

---

## Step-by-Step Practical Workflows

### 1. Clean Testing with Scope Filtering
1. Browse the target app normally with **Intercept is off**.
2. In **Target -> Site map**, right-click your target domain -> **Add to scope**.
3. In **Proxy -> HTTP history**, click the filter bar and check **Show only in-scope items**.

### 2. Testing for IDOR / Broken Access Control in Repeater
1. Log in as User A and view `/api/profile?id=101`.
2. Right-click the request in HTTP history -> **Send to Repeater** (`Ctrl+R`).
3. Change `id=101` to `id=102` -> click **Send**.
4. If the server returns User B's data (`200 OK`) instead of `403 Forbidden`, you found an IDOR.

### 3. Fuzzing Numeric IDs or 4-Digit PINs in Intruder
1. Send request to **Intruder** (`Ctrl+I`).
2. Highlight the parameter value (e.g., `otp=0000`) -> click **Add §**.
3. Select Attack type: **Sniper**.
4. Under **Payloads**, choose **Numbers** (`0000` to `9999`, step `1`) -> **Start attack**.
5. Sort results by **Status code** or **Length** to spot the valid hit.

---

## Common Pitfalls & Tips
- **Leaving Intercept On:** If pages "hang forever" in your browser, check **Proxy -> Intercept**—it's probably waiting for you to click **Forward** or toggle **Intercept off**.
- **Client-Side Only Validation:** Never trust JavaScript checks in the browser; always test the underlying API request in **Repeater**.
