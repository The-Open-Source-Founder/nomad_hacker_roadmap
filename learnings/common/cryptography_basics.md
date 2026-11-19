# Cryptography Basics

> **TL;DR:** Never confuse **Encoding** (reversible format conversion, no key), **Hashing** (one-way fingerprint), and **Encryption** (reversible confidentiality with a key). Use modern algorithms (AES-GCM, ChaCha20, SHA-256, Argon2/bcrypt) and never roll your own crypto.

---

## Core Concepts & Mental Model

| Concept | Reversible? | Requires Key? | Primary Purpose | Examples |
|---------|-------------|---------------|-----------------|----------|
| **Encoding** | Yes (anyone) | No | Data transport / formatting | Base64, Hex, URL encoding |
| **Hashing** | **No** (one-way) | No (salt optional) | Integrity verification, password storage | SHA-256, bcrypt, Argon2id (Avoid MD5/SHA1) |
| **Symmetric Encryption** | Yes | Yes (1 shared key) | Fast bulk data confidentiality | AES-256-GCM, ChaCha20 |
| **Asymmetric Encryption** | Yes | Yes (Public + Private) | Key exchange, digital signatures, TLS | RSA-2048/4096, ECC (Ed25519) |

### How TLS / PKI Works (Simplified)
1. Server sends its **X.509 Certificate** (containing its Public Key, signed by a trusted Certificate Authority).
2. Client verifies the CA signature and domain name.
3. Client & Server use **Asymmetric Crypto** (e.g., ECDHE) to agree on a temporary shared **Session Key**.
4. All subsequent HTTP data is encrypted with fast **Symmetric Crypto** (AES-GCM).

---

## Quick-Reference Command Cheatsheet

### Encoding & Hashing
| Command | Purpose |
|---------|---------|
| `echo -n "hello" \| base64` | Base64 encode a string (`-n` avoids trailing newline) |
| `echo "aGVsbG8=" \| base64 -d` | Base64 decode |
| `sha256sum file.iso` | Compute SHA-256 checksum to verify file integrity |
| `echo -n "password" \| sha256sum` | Hash a string with SHA-256 |

### OpenSSL & GPG
| Command | Purpose |
|---------|---------|
| `openssl rand -hex 32` | Generate a cryptographically secure 256-bit random key |
| `openssl enc -aes-256-cbc -pbkdf2 -in secret.txt -out secret.enc` | Encrypt a file with AES-256 using PBKDF2 key derivation |
| `openssl enc -d -aes-256-cbc -pbkdf2 -in secret.enc -out secret.txt` | Decrypt an AES-256 encrypted file |
| `openssl x509 -in cert.pem -text -noout` | Inspect an X.509 SSL/TLS certificate |
| `gpg --symmetric --cipher-algo AES256 file.txt` | Password-encrypt a file with GPG (`gpg -d` to decrypt) |

---

## Common Pitfalls & Defensive Notes
- **Base64 is NOT Encryption:** Base64 is trivially decoded by anyone; never use it to "hide" secrets or API keys.
- **Fast Hashes for Passwords:** Never store passwords with raw MD5 or SHA-256—GPUs crack billions per second. Always use slow, salted, memory-hard KDFs (**Argon2id**, **bcrypt**, or **scrypt**).
- **ECB Mode Weakness:** AES-ECB encrypts identical blocks into identical ciphertext, leaking patterns. Always use authenticated modes like **AES-GCM**.

---

## Hands-On Lab: Encoding vs. Hashing vs. AES-256 Encryption on Kali

**Goal:** Experience firsthand why Base64 is not security, how hashes change completely with 1 character (avalanche effect), and how to encrypt/decrypt a file with AES-256.

### Step 1: Encoding vs. Hashing (Avalanche Effect)
```bash
mkdir -p /tmp/crypto_lab && cd /tmp/crypto_lab

# 1. Base64 is trivially reversible (NOT encryption!)
echo -n "SuperSecretKey" | base64
echo "U3VwZXJTZWNyZXRLZXk=" | base64 -d; echo

# 2. Hashing is one-way; changing 1 character changes the entire digest
echo -n "Password1" | sha256sum
echo -n "Password2" | sha256sum
```

### Step 2: Symmetric File Encryption with AES-256 + PBKDF2
```bash
echo "Top secret lab notes" > secret.txt

# Encrypt using AES-256-CBC with PBKDF2 key derivation (enter a test passphrase when prompted)
openssl enc -aes-256-cbc -pbkdf2 -salt -in secret.txt -out secret.enc

# View the encrypted binary/bytes (unreadable)
xxd secret.enc | head -n 5

# Decrypt back to plaintext
openssl enc -d -aes-256-cbc -pbkdf2 -in secret.enc -out decrypted.txt
cat decrypted.txt

# Clean up
rm -rf /tmp/crypto_lab
```

---

## Practice & Next Steps
- **Labs:** [CryptoHack](https://cryptohack.org/), TryHackMe *Cryptography for Dummies* & *Encryption - Crypto 101*
- **Next Topic:** [Finding Subdomains & Hidden Paths](finding_subdomain.md)
