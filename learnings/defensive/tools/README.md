# Defensive Track Tools

Detailed, hands-on reference guides for SOC monitoring, packet analysis, SIEM hunting, and DFIR utilities.

| Tool | Domain | What You'll Learn | Guide |
|------|--------|-------------------|-------|
| **Wireshark & TShark** | Network Analysis | Display filters, stream reconstruction, HTTP/SMB object extraction, statistics | [wireshark_and_tshark.md](wireshark_and_tshark.md) |
| **tcpdump** | Packet Capture | BPF capture filters, `-s0` full captures, rotating PCAPs, live CLI triage | [tcpdump.md](tcpdump.md) |
| **Sysmon & Event Logs** | Endpoint Telemetry | Sysmon XML config, high-value Event IDs, `Get-WinEvent` & `wevtutil` queries | [sysmon_and_event_logs.md](sysmon_and_event_logs.md) |
| **Splunk & SIEM (SPL)** | Log Analytics | SPL pipeline anatomy, `stats`/`eval`/`rex`/`bin`, threat-hunting query templates | [splunk_and_siem_queries.md](splunk_and_siem_queries.md) |
| **YARA & Volatility 3** | Malware & Memory DFIR | Writing YARA detection rules, RAM process/network/injection forensics | [yara_and_volatility.md](yara_and_volatility.md) |

[← Back to Master Roadmap](../../../README.md)
