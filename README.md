# 🦈 Wireshark Homelab — Packet Analysis Portfolio

> Live network packet captures across 7 exercises covering core networking protocols.  
> All traffic captured on a real home network (Intel Wi-Fi · BT Hub / Arcadyan router · Linux TTL=64).

[View my html page](https://arturskaufmanis.github.io/Wireshark-packet-analysis/index.html)
---

## 📡 Environment

| Detail | Value |
|---|---|
| Host IP | `192.168.1.65` |
| Gateway | `192.168.1.254` |
| Router | Arcadyan (BT Hub) |
| Router OS | Linux (fingerprinted via TTL=64) |
| Interface | Intel Wi-Fi (promiscuous mode) |
| DNS | `bthub.home` (local resolver, dual-stack) |
| Capture date | April 2026 |

---

## ✅ Exercises Completed (7 / 7)

| # | Exercise | Key Technique |
|---|---|---|
| 01 | Capture Setup & Interface Discovery | BPF capture filters · promiscuous mode · interface selection |
| 02 | ARP & Gateway Discovery | ARP table analysis · passive network mapping |
| 03 | ICMP Ping & Packet Analysis | TTL-based OS fingerprinting · round-trip timing |
| 04 | DNS Queries & NXDOMAIN | Query/response matching · background traffic inventory |
| 05 | TCP 3-Way Handshake | SYN/SYN-ACK/ACK lifecycle · FIN teardown · RST detection |
| 06 | DHCP DORA Handshake | Full IP assignment sequence · transaction ID tracking |
| 07 | HTTP Plaintext Capture | Raw header reading · cookie exposure · security implications |

---

## 🔬 Exercise Highlights

### 01 — Capture Setup
- Identified active Intel Wi-Fi adapter among multiple Hyper-V virtual switches
- Enabled **promiscuous mode** (captures all frames on segment, not just host traffic)
- Learned difference between **capture filters** (BPF, OS-level) and **display filters** (post-capture)
- Recommended buffer size: 64MB+ for busy captures

### 02 — ARP & Gateway Discovery
- `192.168.1.1` did not respond — Wireshark showed only unanswered ARP broadcasts
- Used `arp -a` to confirm true gateway: `192.168.1.254`
- Mapped 5 devices on the subnet via ARP table (`.10`, `.102`, `.217`, `.249`, `.254`)

### 03 — ICMP Ping & OS Fingerprinting
- `ping 192.168.1.254 -n 10` — zero packet loss, 1–3ms round trip
- **TTL=64** in replies confirmed router runs **Linux** (Windows=128, Cisco=255)
- Observed IGMPv3 Membership Query — normal multicast management traffic

### 04 — DNS & NXDOMAIN
- Triggered deliberate NXDOMAIN for `nonexistent.fakedomainxyz.com`
- Both **A and AAAA** records queried automatically (dual-stack behaviour)
- Background DNS passively revealed active services: `ssl.gstatic.com`, `assets.msn.com`, `web.whatsapp.com`
- Captured DNS over **TCP** for large SOA responses (not just UDP)

### 05 — TCP 3-Way Handshake
```
SYN      192.168.1.65  → 192.168.1.254   Seq=0  Win=65535  MSS=1460  SACK_PERM
SYN,ACK  192.168.1.254 → 192.168.1.65    Seq=0  Ack=1      MSS=1460  WS=64
ACK      192.168.1.65  → 192.168.1.254   Seq=1  Ack=1
HTTP GET /nonAuth/wan_conn.xml HTTP/1.1
FIN,ACK  — clean connection close (no RST observed)
```
- Bonus finding: `example.com` served via **Cloudflare London (LHR)** · CF-RAY: `9f3edbe12dd59df1-LHR`

### 06 — DHCP DORA Handshake
| Step | From | To | Time |
|---|---|---|---|
| Release | 192.168.1.65 | 192.168.1.254 | 0.000s |
| Discover | 0.0.0.0 | 255.255.255.255 | 8.332s |
| Offer | 192.168.1.254 | 192.168.1.65 | 8.335s |
| Request | 0.0.0.0 | 255.255.255.255 | 8.379s |
| ACK | 192.168.1.254 | 192.168.1.65 | 8.538s |

- Full DORA completed in **206 milliseconds**
- Transaction ID `0xeebcfcff` consistent across all 4 DORA packets

### 07 — HTTP Plaintext
```
GET /nonAuth/wan_conn.xml HTTP/1.1
Host: 192.168.1.254
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Cookie: urn=102...  ← session token visible in plaintext
```
- BT Hub admin page polled every ~3 seconds via background AJAX
- **Cookie header readable directly from raw packet bytes** — demonstrates why HTTPS matters

---

## 🔧 Filter Reference

### Capture Filters (BPF syntax — applied at OS level)
```
host 192.168.1.254          # Traffic to/from gateway only
port 53                     # DNS traffic
port 67 or port 68          # DHCP traffic
port 80                     # HTTP plaintext
host 192.168.1.254 and tcp  # TCP to gateway only
```

### Display Filters (Wireshark syntax — post-capture)
```
dns                         # Show only DNS packets
dhcp                        # Show only DHCP packets
http                        # Show only HTTP packets
tcp.flags.syn==1            # TCP SYN — new connections only
tcp.flags.reset==1          # TCP RST — connection resets/errors
```

---

## 🧠 Skills Developed

| Skill | Description |
|---|---|
| Capture Filters | BPF syntax applied at OS level before packets reach Wireshark |
| Display Filters | Post-capture Wireshark syntax to isolate protocols and flags |
| Protocol Reading | ARP · ICMP · DNS · TCP · DHCP · HTTP · IGMPv3 |
| OS Fingerprinting | Passive TTL-based OS identification without active scanning |
| Network Mapping | ARP table analysis to inventory devices on a subnet |
| Hex Packet Reading | Extracting plaintext headers and cookies from raw bytes |
| TCP Lifecycle | Handshake · data transfer · FIN teardown · RST detection |
| DNS Analysis | Query/response matching · NXDOMAIN · background traffic inventory |

---

## 📁 Repository Structure

```
wireshark-homelab/
├── index.html        # Interactive portfolio page (open in browser)
└── README.md         # This file
```

---

## 🚀 Viewing the Portfolio

Clone the repo and open `index.html` directly in any browser — no server required.

```bash
git clone https://github.com/<your-username>/wireshark-homelab.git
cd wireshark-homelab
open index.html   # macOS
# or double-click index.html on Windows/Linux
```

---

*lijab · 2026 · live network captures on a real home network*
