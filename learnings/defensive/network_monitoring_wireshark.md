# Network Monitoring & Wireshark (Defensive)

> **TL;DR:** Packet captures (PCAPs) are ground truth on the network. Use `tcpdump` or `tshark` for fast CLI captures/slicing, **Wireshark display filters** to isolate suspicious HTTP/DNS/SMB/C2 traffic, and **Suricata/Snort** rules for automated IDS detection.

---

## Core Concepts & Mental Model

- **Full Packet Capture (PCAP)** records every byte of traffic (payload + headers), whereas **NetFlow / Zeek logs** record metadata (who talked to whom, ports, bytes, duration) at much lower storage cost.
- **What Suspicious Traffic Looks Like:**
  - **Port Scan:** Rapid `SYN` packets from 1 IP to many ports, followed by `RST` responses.
  - **C2 Beaconing:** Periodic outbound HTTPS/DNS connections at regular intervals (e.g., every 60s +/- jitter) to a newly registered domain.
  - **DNS Tunneling:** Unusually long, high-entropy subdomains (`aGVsbG8td29ybGQ...evil.com`) and high volume of `TXT` queries.
  - **Cleartext Credentials:** HTTP Basic Auth, FTP `USER`/`PASS`, Telnet, or unencrypted LDAP binds.

---

## Quick-Reference Command & Filter Cheatsheet

### 1. Essential Wireshark Display Filters
| Filter | What It Shows |
|--------|---------------|
| `ip.addr == 10.10.10.5` | All traffic to or from `10.10.10.5` |
| `tcp.port == 4444` | Traffic on port `4444` (common reverse shell port) |
| `http.request.method == "POST"` | Only HTTP `POST` requests (logins, uploads, form submissions) |
| `dns.qry.name contains "evil"` | DNS queries matching a substring |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Initial TCP `SYN` packets (spot port scans) |
| `smb2 \|\| ntlmssp` | SMBv2 traffic and NTLM authentication handshakes |
| `!(arp \|\| dns \|\| icmp)` | Hide noisy background protocols |

*Tip:* Right-click any suspicious packet -> **Follow -> TCP Stream** (or **HTTP Stream**) to reconstruct the full conversation.

### 2. `tcpdump` & `tshark` CLI Cheatsheet
| Command | Purpose |
|---------|---------|
| `sudo tcpdump -i eth0 -w capture.pcap` | Capture all packets on `eth0` and write to `capture.pcap` |
| `sudo tcpdump -i eth0 host 10.10.10.5 and port 80 -nn` | Live capture for specific host & port without DNS resolution |
| `tshark -r capture.pcap -Y "http.request" -T fields -e ip.src -e http.host -e http.request.uri` | Extract HTTP request source IP, host, and URI from a PCAP |
| `tshark -r capture.pcap -q -z conv,tcp` | Show TCP conversation summary (top talkers & byte counts) |
| `tshark -r capture.pcap --export-objects http,./extracted/` | Extract all files transferred over HTTP from a PCAP |

### 3. Basic Suricata / Snort IDS Rule Anatomy
```text
alert http any any -> $HOME_NET any (msg:"Possible SQLi UNION SELECT"; content:"UNION SELECT"; nocase; sid:1000001; rev:1;)
```

---

## Common Pitfalls & Defensive Notes
- **Capture Filter vs. Display Filter:** `tcpdump` uses **BPF syntax** (`host 1.2.3.4 and port 80`), whereas Wireshark uses **Display Filter syntax** (`ip.addr == 1.2.3.4 && tcp.port == 80`). Don't mix them up.
- **Encrypted Traffic (TLS):** Wireshark cannot read HTTPS payloads without TLS session keys (`SSLKEYLOGFILE`). Use **SNI** (`tls.handshake.extensions_server_name`), JA3/JA4 fingerprints, and certificate fields to inspect encrypted connections.

---

## Hands-On Lab: Capture Live HTTP Traffic & Carve Files with `tcpdump` & `tshark`

**Goal:** Generate simulated HTTP traffic locally, capture it to a `.pcap` with `tcpdump`, and extract the HTTP request and transferred file using `tshark` on Kali.

### Step 1: Start a Local HTTP Server & Capture on Loopback (`lo`)
```bash
mkdir -p /tmp/pcap_lab/served /tmp/pcap_lab/carved
echo "CONFIDENTIAL_DATA_EXFIL_TEST" > /tmp/pcap_lab/served/secret.txt

# Start HTTP server in background
python3 -m http.server 9999 --directory /tmp/pcap_lab/served &
HTTP_PID=$!

# Capture 30 packets on loopback port 9999 in background
sudo tcpdump -i lo -nn -s 0 port 9999 -c 30 -w /tmp/pcap_lab/traffic.pcap &
TCPDUMP_PID=$!
sleep 1

# Simulate client downloading secret.txt
curl -s http://127.0.0.1:9999/secret.txt > /dev/null
sleep 1
kill $HTTP_PID 2>/dev/null || true
wait $TCPDUMP_PID 2>/dev/null || true
```

### Step 2: Analyze & Carve the Exfiltrated File with `tshark`
```bash
# 1. List HTTP requests in the PCAP
tshark -r /tmp/pcap_lab/traffic.pcap -Y "http.request" -T fields -e ip.src -e http.request.method -e http.request.uri

# 2. Carve the transferred file straight out of the PCAP
tshark -r /tmp/pcap_lab/traffic.pcap --export-objects http,/tmp/pcap_lab/carved/
cat /tmp/pcap_lab/carved/secret.txt

# Clean up
rm -rf /tmp/pcap_lab
```
*Teaching Moment:* Even without access to the server or client, a network defender with a PCAP can reconstruct the exact URI requested and carve the exact file transferred over unencrypted HTTP!

---

## Practice & Next Steps
- **Labs:** TryHackMe *Wireshark: The Basics*, *Wireshark: Packet Operations*, *Traffic Analysis*, [Malware-Traffic-Analysis.net](https://www.malware-traffic-analysis.net/)
- **Next Topic:** [Log Analysis & SIEM](log_analysis_and_siem.md)
