# Wireshark & TShark — Deep Packet Inspection & PCAP Analysis

> **TL;DR:** **Wireshark** (GUI) and **TShark** (CLI) dissect network packet captures (PCAPs) down to individual bytes. Use Wireshark for visual stream reconstruction and statistics, and TShark for automated extraction and large-PCAP filtering.

---

## Installation & Availability
Pre-installed on Kali Linux. Cross-platform for Windows, macOS, and Linux.
```bash
sudo apt install -y wireshark tshark
```

---

## 1. Wireshark Display Filters Reference

*(Note: Display filters use `==`, `&&`, `||`, `!` — different from `tcpdump` BPF capture filters!)*

| Display Filter | What It Isolates |
|----------------|------------------|
| `ip.addr == 10.10.10.5` | Traffic where source OR destination is `10.10.10.5` |
| `ip.src == 10.10.10.5 && tcp.dstport == 443` | Outbound HTTPS traffic from a specific host |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Initial TCP `SYN` packets (spot port scans) |
| `http.request.method == "POST"` | HTTP `POST` requests (logins, data exfil, file uploads) |
| `http.response.code >= 400` | HTTP errors (`403`, `404` fuzzing, `500` server errors) |
| `dns.qry.name contains "evil"` | DNS queries matching a suspicious domain |
| `smb2 \|\| ntlmssp` | SMB file sharing and NTLM authentication challenges |
| `tcp.analysis.retransmission` | Network drops or scanning anomalies |
| `frame contains "password"` | Case-sensitive payload search across all packets |

### Essential Wireshark GUI Features
- **Follow Stream:** Right-click packet -> **Follow -> TCP Stream** (or **HTTP Stream**) to read the entire conversation in plain text.
- **Export Objects:** **File -> Export Objects -> HTTP / SMB** to carve transferred executables, scripts, or documents straight out of a PCAP!
- **Statistics -> Endpoints / Conversations:** Instantly identify top talkers, suspicious external IPs, and data exfiltration volumes.

---

## 2. TShark CLI Workflows

| Command | Purpose |
|---------|---------|
| `tshark -r capture.pcap -q -z io,phs` | Protocol Hierarchy Statistics (quick overview of what's inside a PCAP) |
| `tshark -r capture.pcap -q -z conv,ip` | List all IP conversations sorted by bytes transferred |
| `tshark -r capture.pcap -Y "dns.qry.name" -T fields -e ip.src -e dns.qry.name` | Extract every DNS query and source IP |
| `tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri -e http.user_agent` | List all HTTP hosts, URIs, and User-Agents |
| `tshark -r capture.pcap -Y "tcp.stream eq 3" -w stream3.pcap` | Carve a single TCP stream into a new PCAP |
| `tshark -r capture.pcap --export-objects http,./carved_files/` | Extract all HTTP-transferred files from CLI |

---

## Common Pitfalls & Defensive Notes
- **Analyzing Untrusted PCAPs:** Wireshark dissectors parse complex protocols and have historically had CVEs. Keep Wireshark updated and analyze unknown PCAPs inside an isolated VM.
- **Encrypted HTTPS:** You won't see HTTP bodies for TLS traffic without server private keys (non-PFS) or `SSLKEYLOGFILE`. Instead, inspect **DNS queries**, **TLS SNI** (`tls.handshake.extensions_server_name`), and **JA3/JA4** fingerprints.
