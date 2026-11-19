# Reconnaissance & OSINT (Offensive)

> **TL;DR:** Reconnaissance is gathering intelligence about a target **before** touching its systems directly (Passive OSINT) or lightly probing public surfaces (Active Recon). Good recon often uncovers exposed credentials, forgotten assets, and tech stacks without firing a single exploit.

---

## Core Concepts & Mental Model

| Type | Definition | Detection Risk | Examples |
|------|------------|----------------|----------|
| **Passive Recon** | Gathering data from third-party/public sources without sending packets to the target | **Zero** (undetectable by target) | WHOIS, Shodan, Google Dorks, `crt.sh`, GitHub leaks, LinkedIn |
| **Active Recon** | Directly interacting with target infrastructure | **Low–Medium** (logged by target) | DNS queries, `curl` banner grabbing, `nmap` host discovery |

---

## Quick-Reference Command & Query Cheatsheet

### Google Dorking (Search Engine Operators)
| Dork | What It Finds |
|------|---------------|
| `site:target.com filetype:pdf` | Public PDFs (often leak internal usernames/software in metadata) |
| `site:target.com intitle:"index of"` | Open directory listings |
| `site:target.com inurl:admin \| inurl:login` | Exposed login/admin portals |
| `site:github.com "target.com" "API_KEY" \| "password"` | Leaked secrets on GitHub |

### Shodan & Censys Search Filters
| Query | Purpose |
|-------|---------|
| `org:"Target Corp" port:22,3389` | Find exposed SSH/RDP belonging to an organization |
| `ssl:"target.com" http.status:200` | Find servers presenting the target's SSL cert |
| `hostname:target.com product:"Apache"` | Identify web server versions indexed by Shodan |

### CLI OSINT Tools
| Tool / Command | Purpose & Example |
|----------------|-------------------|
| `whois target.com` | Registrar, creation/expiry dates, and name servers |
| `dig +short MX TXT NS target.com` | Enumerate mail servers (MX), SPF/DMARC records (TXT), and nameservers |
| `theHarvester -d target.com -b crtsh,duckduckgo,bing` | Harvest emails, subdomains, and hosts from public sources |
| `exiftool document.pdf` | Extract author names, software versions, and GPS metadata from downloaded files |
| `whatweb https://target.com` | Fingerprint web technologies, CMS, frameworks, and server headers |

---

## Common Pitfalls & Defensive Notes
- **Scope Creep:** Passive OSINT often reveals third-party SaaS vendors or partner domains. Never actively scan anything outside your authorized **Rules of Engagement (RoE)**.
- **Defensive Tip:** Periodically run Google Dorks and Shodan queries against your own organization's domain, strip EXIF metadata from public documents, and use secret-scanning hooks (e.g., `gitleaks`) in CI/CD.

---

## Hands-On Lab: Passive OSINT & Metadata Extraction on Kali

**Goal:** Run safe, passive DNS/certificate lookups on a public test domain (`example.com`) and extract hidden metadata from a sample file using `exiftool`.

### Step 1: Passive DNS & Certificate Transparency Enumeration
```bash
# 1. Check authoritative nameservers and mail servers
dig example.com NS +short
dig example.com MX +short
dig example.com TXT +short

# 2. Query Certificate Transparency logs (crt.sh) for subdomains
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sort -u | head -n 20
```
*Teaching Moment:* None of these commands touch `example.com`'s web servers directly—`crt.sh` is a public third-party certificate log, making this **100% passive**.

### Step 2: Practice Metadata Extraction & Stripping with `exiftool`
```bash
# Create a dummy text/image or inspect any downloaded PDF/image
curl -s https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf -o /tmp/sample.pdf
exiftool /tmp/sample.pdf
```
*What to look for:* `Author`, `Creator`, `Producer` (reveals exact OS/software versions like LibreOffice or Adobe Acrobat). Clean up with `rm /tmp/sample.pdf`.

---

## Practice & Next Steps
- **Labs:** TryHackMe *Passive Reconnaissance*, *Active Reconnaissance*, *Searchlight - IMINT*
- **Next Topic:** [Scanning & Enumeration](scanning_and_enumeration.md)
