# OpenSSL — TLS Inspection, Certificates & Cryptography CLI

> **TL;DR:** `openssl` is the standard CLI toolkit for inspecting SSL/TLS connections, decoding X.509 certificates, generating keys/CSRs, hashing, and encrypting/decrypting files.

---

## Installation & Availability
Pre-installed on nearly all Linux distros and macOS.
```bash
sudo apt install -y openssl
```

---

## Essential Subcommands Reference

| Subcommand | Purpose |
|------------|---------|
| `openssl s_client` | Connect to a TLS/SSL server to inspect certificates, ciphers, and protocol versions |
| `openssl x509` | View, convert, or verify X.509 digital certificates |
| `openssl enc` | Symmetric file encryption and decryption (e.g., AES-256-CBC) |
| `openssl rand` | Generate cryptographically secure random bytes (Hex or Base64) |
| `openssl genrsa` / `genpkey` | Generate RSA or Ed25519 private keys |
| `openssl dgst` | Compute cryptographic hashes (SHA-256, SHA-512) |

---

## Step-by-Step Practical Workflows

### 1. Inspect a Website's Live TLS Certificate & Cipher
```bash
openssl s_client -connect example.com:443 -servername example.com -brief
```
To extract **Subject Alternative Names (SANs)** (often reveals extra internal/staging domains!):
```bash
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null \
  | openssl x509 -noout -text | grep -A1 "Subject Alternative Name"
```

### 2. Inspect a Local `.pem` or `.crt` Certificate File
```bash
openssl x509 -in server.crt -noout -subject -issuer -dates
```

### 3. Generate a Self-Signed Certificate (For Lab HTTPS Listeners)
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=lab.local"
```

### 4. Encrypt & Decrypt a File Securely with AES-256
Always include `-pbkdf2` for proper key derivation:
```bash
# Encrypt
openssl enc -aes-256-cbc -pbkdf2 -salt -in secrets.txt -out secrets.enc
# Decrypt
openssl enc -d -aes-256-cbc -pbkdf2 -in secrets.enc -out secrets.txt
```

### 5. Generate Secure Random Tokens / Keys
```bash
openssl rand -hex 32      # 256-bit hex key
openssl rand -base64 24   # Random Base64 password/token
```

---

## Common Pitfalls & Defensive Notes
- **Missing `-servername` (SNI):** When testing HTTPS on shared IPs/CDNs, omitting `-servername <domain>` may return a default/wrong certificate.
- **Weak TLS Protocols:** Test if a server still accepts deprecated TLS 1.0/1.1 using `-tls1` or `-tls1_1`; production servers should only allow **TLS 1.2** and **TLS 1.3**.
