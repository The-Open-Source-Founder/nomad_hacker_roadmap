# dig & DNS Tools — DNS Enumeration & Diagnostics

> **TL;DR:** `dig` (Domain Information Groper) queries DNS servers for `A`, `AAAA`, `MX`, `TXT`, `NS`, and `SOA` records, tests specific nameservers, and attempts DNS zone transfers (`AXFR`).

---

## Installation & Availability
Included in `dnsutils` (Debian/Ubuntu/Kali) or `bind-utils` (RHEL/Fedora).
```bash
sudo apt install -y dnsutils
```

---

## Essential Syntax & Flags Reference

```bash
dig [@nameserver] <domain> [record_type] [options]
```

| Option / Type | Description |
|---------------|-------------|
| `@8.8.8.8` | Query a specific DNS server instead of your OS default |
| `A` / `AAAA` | IPv4 / IPv6 address records |
| `MX` | Mail Exchange servers (reveals email provider & potential mail relays) |
| `NS` | Authoritative Name Servers for the domain |
| `TXT` | Text records (often leak SPF, DMARC, domain verification tokens, cloud services) |
| `CNAME` | Canonical Name (alias; useful for spotting subdomain takeover candidates) |
| `ANY` | Request all available record types (blocked by some servers) |
| `AXFR` | Request a full **DNS Zone Transfer** |
| `-x <IP>` | Reverse DNS lookup (PTR record: IP -> hostname) |
| `+short` | Print only the clean answer value without headers/stats |
| `+trace` | Trace resolution path from root `.` servers down to the authoritative NS |

---

## Step-by-Step Practical Workflows

### 1. Gather Full DNS Profile of a Target
```bash
dig target.com NS +short
dig target.com MX +short
dig target.com TXT +short
```
*Why `TXT` matters:* `v=spf1 include:_spf.google.com ...` or `MS=ms123456` tells you the organization uses Google Workspace or Microsoft 365.

### 2. Attempt a DNS Zone Transfer (`AXFR`)
If an authoritative nameserver is misconfigured to allow zone transfers to any client, it dumps **every** internal/external DNS record in the zone:
```bash
# 1. Find authoritative nameservers
dig target.com NS +short
# 2. Test AXFR against each nameserver
dig @ns1.target.com target.com axfr
```

### 3. Reverse DNS Lookup (PTR)
```bash
dig -x 10.10.10.5 +short
```

### 4. Query Internal AD DNS from Kali
When testing an internal network or Active Directory lab where the Domain Controller (`10.10.10.10`) is the DNS server:
```bash
dig @10.10.10.10 dc01.corp.local A +short
dig @10.10.10.10 _ldap._tcp.dc._msdcs.corp.local SRV +short
```

### 5. Quick Alternatives (`host` & `nslookup`)
| Command | Purpose |
|---------|---------|
| `host -t txt target.com` | Clean, concise TXT record lookup |
| `host -l target.com ns1.target.com` | Attempt zone transfer using `host` |
| `nslookup -type=mx target.com 8.8.8.8` | Cross-platform (works natively on Windows & Linux) |

---

## Common Pitfalls & Defensive Notes
- **`ANY` Queries Blocked:** Modern DNS providers (Cloudflare, Route53) often refuse `ANY` queries (`RFC 8482`). Query specific record types (`A`, `MX`, `TXT`, `NS`) individually.
- **Defensive Tip:** Restrict `AXFR` zone transfers strictly to authorized secondary nameserver IPs (`allow-transfer { <secondary_ip>; };` in BIND), and audit `TXT` records so internal infrastructure details aren't exposed publicly.
