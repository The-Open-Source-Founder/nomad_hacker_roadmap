# Kali OSINT & Social Engineering Tools (`theHarvester`, `sherlock`, `exiftool`, `setoolkit`)

> **TL;DR:** Before launching technical exploits, attackers map people, emails, leaked metadata, and social attack surfaces. Kali provides **`theHarvester`** (email/subdomain OSINT), **`sherlock`** (username hunting), **`exiftool`** (file metadata), and **`setoolkit`** (authorized phishing/awareness simulations).

---

## Installation & Availability in Kali Linux
```bash
sudo apt install -y theharvester sherlock libimage-exiftool-perl set
```

---

## 1. `theHarvester` — Email, Subdomain & Host OSINT

Queries search engines, Certificate Transparency (`crtsh`), DuckDuckGo, and public APIs:
```bash
theHarvester -d target.com -b crtsh,duckduckgo,bing,yahoo -l 200
```
| Flag | Purpose |
|------|---------|
| `-d <domain>` | Target domain |
| `-b <sources>` | Data sources (`all`, `crtsh`, `duckduckgo`, `bing`, `shodan`) |
| `-l <limit>` | Max results per source |
| `-f <file>` | Save results to `.json` and `.xml` |

---

## 2. `sherlock` — Username Enumeration Across Social Platforms
Useful for correlating a target's handle across 300+ websites during authorized OSINT investigations:
```bash
sherlock target_username --timeout 5 -o sherlock_results.txt
```

---

## 3. `exiftool` — Document & Image Metadata Extraction
Public PDFs, DOCX files, and images frequently leak **internal usernames**, **software versions**, **printers**, and **GPS coordinates**:
```bash
exiftool downloaded_report.pdf
# Strip all metadata before publishing your own files:
exiftool -all= clean_report.pdf
```

---

## 4. `setoolkit` — Social-Engineer Toolkit (Authorized Simulations Only)
Maintained by TrustedSec and pre-installed on Kali (`sudo setoolkit`), `setoolkit` automates authorized security awareness & phishing simulations:
- **Credential Harvester Attack Method:** Clones a login page to test whether employees recognize spoofed internal portals during approved awareness drills.
- **Spear-Phishing Attack Vectors:** Tests email filtering and user reporting workflows.

---

## Common Pitfalls & Defensive Notes
- **Unauthorized Phishing / Harassment:** Never run `setoolkit` or targeted credential harvesting outside a signed, scoped security awareness engagement.
- **Defensive Tip:** Strip EXIF metadata automatically from all public uploads/documents, enforce **FIDO2/WebAuthn hardware MFA** (phishing-resistant!), and configure **DMARC (`p=reject`) + SPF + DKIM** to block domain spoofing.
