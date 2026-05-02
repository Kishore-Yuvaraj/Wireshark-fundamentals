# Wireshark Fundamentals — Packet Analysis Mastery

A hands-on learning series where I capture real network traffic, analyze live protocols (DNS, HTTP, TCP, TLS), simulate real attacks in an isolated lab, and document findings like a SOC analyst.

---

## 🎯 Quick Overview

**Goal:** Master Wireshark packet analysis from real network captures — moving from theory to practical SOC analyst skills.

**Current Status:**

| Session | Topic | Status |
|---------|-------|--------|
| Session 1 | Live Traffic Capture — DNS, HTTP, TCP Handshake | ✅ Complete |
| Session 2 | TLS/HTTPS Deep Dive — Certificate Analysis | ✅ Complete |
| Session 3A | ARP Spoofing Live Lab Simulation | ✅ Complete |
| Session 3B | DNS Tunneling Detection | 🔜 Coming Soon |
| Session 4 | CTF-Style Mystery PCAP Challenge | 🔜 Coming Soon |
| Session 5 | Resume Portfolio + Cheatsheet | 🔜 Coming Soon |

---

## 📁 What's in This Repo

```
wireshark-fundamentals/
├── README.md (this file)
│
├── session-1/
│   ├── wireshark-session1-writeup.md
│   └── screenshots/
│       ├── tcp-handshake-packets.png
│       ├── dns-response.png
│       └── http-redirect-flow.png
│
├── session-2/
│   ├── wireshark-session2-writeup.md
│   ├── TLS-Certificate-Inspection-SOC-Guide.md
│   ├── Tls_HandShake.pcapng (5,538 real packets)
│   └── screenshots/
│       └── tcp-stream-encrypted.png
│
└── session-3a-arp-spoofing/
    ├── arp-spoofing-writeup.md
    ├── arp-capture.pcap
    └── screenshots/
        ├── 01-kali-arpspoof-running.png
        ├── 02-wireshark-arp-packets.png
        ├── 03-packet-dissection-fields.png
        └── 04-ubuntu-arp-table-poisoned.png
```

---

## 🚀 Session 1 — Live Traffic Capture & Protocol Fundamentals

**Capture**
- Launched Wireshark on Windows
- Visited google.com, github.com, youtube.com in browser
- Captured 6,688 packets in ~30 seconds

**Analyze**
- **DNS:** Filtered by `dns` → Found query for dns.google → Response: 8.8.4.4, 8.8.8.8
- **HTTP:** Filtered by `http` → Traced `GET /online` → 301 redirect → 200 OK response
- **TCP:** Filtered by `tcp.flags.syn==1` → Found 3-packet handshake (SYN → SYN,ACK → ACK)

**Key Finding**

All traffic was legitimate — normal TTLs, clean handshake, expected redirects. No indicators of compromise.

📖 Full write-up: [session-1/wireshark-session1-writeup.md](session-1/wireshark-session1-writeup.md)

---

## 🔐 Session 2 — TLS/HTTPS Deep Dive & Certificate Analysis

**Capture**
- Launched Wireshark on Windows with Edge browser
- Visited google.com, github.com over HTTPS
- Captured 5,538 packets — 1,005 TLS packets identified

**Analyze**
- **TLS Handshake:** Found Client Hello (packet 159) and Server Hello (packet 162)
- **Certificate Extraction:** Extracted Google certificate from browser (TLS 1.3 encrypts it in Wireshark)
- **Certificate Validation:** Verified issuer (Google Trust Services), validity dates, SHA-256 fingerprint
- **Cipher Suite:** Identified `TLS_AES_256_GCM_SHA384` — military-grade encryption ✅
- **TCP Stream:** Used Follow TCP Stream — confirmed encrypted gibberish = encryption working correctly

**Key Finding**

All HTTPS traffic was legitimate — valid certificate, strong cipher, encrypted data confirmed. No MITM indicators.

📖 Full write-up: [session-2/wireshark-session2-writeup.md](session-2/wireshark-session2-writeup.md)
📋 Reference guide: [session-2/TLS-Certificate-Inspection-SOC-Guide.md](session-2/TLS-Certificate-Inspection-SOC-Guide.md)

---

## ⚔️ Session 3A — ARP Spoofing Live Lab Simulation

> **First attack simulation session** — Instead of analyzing pre-made traffic, I built an isolated lab from scratch and generated real attack traffic to analyze.

### Lab Environment

| Machine | OS | IP Address | Role |
|---------|-----|------------|------|
| Attacker | Kali Linux | 192.168.100.10 | ARP Spoofer |
| Victim | Ubuntu 24.04 | 192.168.100.50 | Target |
| Network | VirtualBox Internal Network ("labnet") | — | Fully Isolated |

### What Was Done

1. Configured two VMs on an isolated VirtualBox internal network with static IPs
2. Ran a live ARP spoofing attack from Kali using `arpspoof`
3. Captured all traffic with `tcpdump` and opened the PCAP in Wireshark
4. Identified 3 forensic indicators of an active ARP spoofing attack
5. Verified ARP table poisoning on the Ubuntu victim with `ip neigh show`

### Attack Command Used

```bash
sudo arpspoof -i eth0 -t 192.168.100.50 192.168.100.1
```

### Screenshot — Wireshark Capture of the Attack

![Wireshark ARP Spoofing Capture](session-3a-arp-spoofing/screenshots/02-wireshark-arp-packets.png)

*The Delta column shows ~2 second intervals between ARP replies — the automated timing signature of arpspoof.*

### 3 Forensic Indicators Identified (SOC Perspective)

| # | Indicator | What It Means |
|---|-----------|---------------|
| 1 | **~2 second intervals in Delta column** | Automated spoofing tool — humans don't send ARP at machine precision |
| 2 | **Gratuitous ARP replies (`arp.opcode == 2`)** | Unsolicited replies overwriting the victim's ARP cache |
| 3 | **ARP = 65.1% of all captured traffic** | Abnormal ARP flood — legitimate networks have <5% ARP traffic |

### Key Wireshark Filters Used

```
arp                  # all ARP traffic
arp.opcode == 2      # ARP replies only — detect gratuitous ARP
```

### Key Finding

ARP table poisoning confirmed on victim machine. The three forensic indicators together form a definitive signature of an active ARP spoofing attack — not a false positive.

📖 Full write-up: [session-3a-arp-spoofing/arp-spoofing-writeup.md](session-3a-arp-spoofing/arp-spoofing-writeup.md)

---

## 💡 What This Proves

| Skill | Evidence |
|-------|----------|
| ✅ Live traffic capture | Sessions 1 & 2 — real packets from my own machine |
| ✅ Protocol analysis | DNS, HTTP, TCP, TLS handshakes analyzed from real captures |
| ✅ SOC analyst mindset | Certificate validation, MITM detection, IoC identification |
| ✅ Attack simulation | Session 3A — built isolated lab, executed real ARP spoofing attack |
| ✅ Forensic detection | Identified 3 indicators from raw PCAP — timing, gratuitous ARP, flood % |
| ✅ Lab documentation | Full write-ups, PCAP files, screenshots committed to GitHub |

---

## 🛠️ Wireshark Filters Cheatsheet

```
# Protocol filtering
dns                          # DNS packets only
http                         # HTTP packets only
tcp                          # TCP packets only
tls                          # TLS/HTTPS packets only
arp                          # ARP packets only

# Connection starts
tcp.flags.syn==1             # SYN packets
tcp.flags.syn==1 AND tcp.dstport==443  # HTTPS attempts

# TLS specific
tls.handshake.type==1        # Client Hello only
tls.handshake.type==2        # Server Hello only

# ARP attack detection
arp.opcode == 2              # ARP replies (look for gratuitous ARP)

# Filter by IP
ip.src==192.168.1.100        # From this IP
ip.dst==8.8.8.8              # To this IP

# Filter by port
tcp.port==443                # HTTPS
tcp.port==22                 # SSH
tcp.dstport==80              # HTTP incoming
```

---

## ⚠️ Red Flags — What to Look For in SOC Work

| Protocol | Red Flag | Why It Matters |
|----------|----------|----------------|
| DNS | TTL < 60 seconds | Malware rotates IPs rapidly (fast-flux) |
| DNS | Query to random-string domain | Typosquatting, phishing, DGA malware |
| HTTP | 3+ consecutive redirects | Phishing chain, malware delivery |
| HTTP | GET to unusual path (e.g., `/admin.php?cmd=`) | Possible exploitation attempt |
| TCP | SYN to unusual port (31337, 6666) | Backdoor activity |
| TCP | Many SYN with no SYN,ACK | Port scan or DoS |
| TLS | Self-signed certificate | No legitimate CA — likely attacker |
| TLS | Expired certificate | Misconfigured or compromised server |
| TLS | Weak cipher (RC4, DES, MD5) | Vulnerable to cryptographic attacks |
| ARP | Gratuitous ARP replies (`arp.opcode == 2`) | ARP spoofing / MITM attack |
| ARP | ARP > 5% of total traffic | Abnormal — possible ARP flood |
| ARP | Fixed ~2s interval ARP replies | Automated spoofing tool signature |

---

## 🔥 For Job Interviews

**When asked "How would you analyze a PCAP file?"**

> "I start with Protocol Hierarchy (`Statistics → Hierarchy`) to see what protocols are present and their proportions. Abnormal ratios — like ARP being 65% of traffic — are immediate red flags. For HTTP I trace redirect chains and check for unusual paths. For DNS I look for fast-flux indicators like TTL under 60 seconds. For HTTPS, I filter by TLS and find the Client Hello to check the SNI, then the Server Hello for cipher suite strength. I extract the certificate and verify: subject matches domain, issuer is a trusted CA, certificate is currently valid. I use Follow TCP Stream to confirm encryption. For ARP, I filter `arp.opcode == 2` and check the Delta column for machine-precision intervals — a hallmark of automated spoofing tools. Here's a real example from my Session 3A lab where I executed a live ARP spoofing attack and identified all three forensic indicators in the PCAP."

---

## 📊 Session Results Summary

| Session | Packets Captured | Protocols | Verdict |
|---------|-----------------|-----------|---------|
| Session 1 | 6,688 | DNS, HTTP, TCP | Legitimate — no IoCs |
| Session 2 | 5,538 | TLS, HTTPS, TCP | Legitimate — valid certificate, strong cipher |
| Session 3A | 36 (attack traffic) | ARP | ⚠️ Attack confirmed — 3 IoCs identified |

---

## 🔄 Next Steps

- **Session 3B:** DNS Tunneling — detect covert data exfiltration via DNS queries
- **Session 3C:** SYN Flood + Data Exfiltration — simulate TCP SYN flood attack and identify covert data exfiltration through abnormal traffic patterns
- **Session 4:** CTF-style challenge — given a malicious PCAP, extract IoCs and write incident report
- **Session 5:** Build final cheatsheet + portfolio summary

---

## 📚 References

- [Wireshark Official Documentation](https://www.wireshark.org/docs/)
- My learning path: Cisco Intro to Cybersecurity → TCM Linux 100 → TryHackMe labs → Wireshark hands-on series

---

*Kishore Yuvaraj | github.com/Kishore-Yuvaraj*
