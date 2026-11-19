# tcpdump — Command-Line Packet Capture

> **TL;DR:** `tcpdump` is installed on nearly every Linux server and firewall, making it indispensable during live incident response when a GUI isn't available. Capture traffic with **BPF filters** and write to `.pcap` (`-w`) for deeper analysis in Wireshark/TShark.

---

## Installation & Availability
```bash
sudo apt install -y tcpdump
```

---

## Essential Flags Reference

| Flag | Description |
|------|-------------|
| `-i <iface>` | Network interface to listen on (e.g., `eth0`, `tun0`, or `any`) |
| `-nn` | Disable both DNS (`-n`) and port-name (`-nn`) resolution (faster & avoids DNS noise) |
| `-w <file.pcap>` | Write raw packets to a PCAP file instead of printing to terminal |
| `-r <file.pcap>` | Read and filter packets from an existing PCAP file |
| `-s 0` | Snaplen `0` = capture **full** packets (default on modern `tcpdump`, good habit to specify) |
| `-c <count>` | Stop automatically after capturing `<count>` packets |
| `-A` / `-X` | Print packet payloads in ASCII (`-A`) or Hex + ASCII (`-X`) |
| `-C <MB>` / `-W <n>` | Rotate capture files every `<MB>` megabytes, keeping `<n>` files (prevents filling disk!) |

---

## Step-by-Step Practical Workflows

### 1. Safe Rotating Capture During Incident Response
Never leave an unbounded `tcpdump` running on a production server or you'll fill `/`!
```bash
sudo tcpdump -i eth0 -s 0 -nn -C 100 -W 5 -w /tmp/ir_capture.pcap
```
*(Creates up to 5 rotating 100 MB files: `ir_capture.pcap0` .. `ir_capture.pcap4`.)*

### 2. Capture Traffic for a Specific Host & Port (BPF Syntax)
```bash
sudo tcpdump -i eth0 -nn host 10.10.10.5 and port 443 -w https_target.pcap
```

### 3. Live Triage: Watch Cleartext HTTP or DNS Queries
```bash
# Watch DNS queries live
sudo tcpdump -i eth0 -nn udp port 53

# Watch HTTP payloads in ASCII
sudo tcpdump -i eth0 -nn -A tcp port 80
```

### 4. Filter an Existing PCAP to Exclude Noisy SSH Traffic
```bash
tcpdump -r big_capture.pcap -nn "not port 22" -w no_ssh.pcap
```

### 5. Spot TCP SYN Floods or Port Scans Live
```bash
sudo tcpdump -i eth0 -nn "tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0"
```

---

## Common Pitfalls & Defensive Notes
- **Forgetting `-nn`:** Without `-nn`, `tcpdump` performs reverse DNS lookups for every captured IP—slowing down capture and generating outbound DNS queries that tip off attackers or pollute logs.
- **Disk Exhaustion:** Always use `-c` (packet count) or `-C` + `-W` (file rotation) on production servers.
