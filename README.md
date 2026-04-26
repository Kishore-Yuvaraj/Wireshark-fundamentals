# Wireshark Fundamentals — Packet Analysis Mastery

A hands-on learning series where I capture real network traffic, analyze live protocols (DNS, HTTP, TCP, TLS), and document findings like a SOC analyst.

---

## 🎯 Quick Overview

**Goal:** Master Wireshark packet analysis from real network captures — moving from theory to practical SOC analyst skills.

**Current Status:**
- ✅ Session 1 — Live traffic capture & protocol fundamentals (COMPLETE)
- ✅ Session 2 — TLS/HTTPS Deep Dive & Certificate Analysis (COMPLETE)
- ⏳ Session 3–5 — Upcoming (attack traffic, CTF challenge, portfolio)

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
└── session-2/
    ├── wireshark-session2-writeup.md  ← READ THIS FIRST
    ├── TLS-Certificate-Inspection-SOC-Guide.md
    ├── Tls_HandShake.pcapng (5,538 real packets)
    └── screenshots/
        └── tcp-stream-encrypted.png
```

---

## 🚀 Session 1 — Live Traffic Capture & Protocol Fundamentals

### Capture
- Launched Wireshark on Windows
- Visited google.com, github.com, youtube.com in browser
- Captured **6,688 packets** in ~30 seconds

### Analyze
- **DNS:** Filtered by `dns` → Found query for dns.google → Response: 8.8.4.4, 8.8.8.8
- **HTTP:** Filtered by `http` → Traced GET /online → 301 redirect → 200 OK response
- **TCP:** Filtered by `tcp.flags.syn==1` → Found 3-packet handshake (SYN → SYN,ACK → ACK)

### Key Finding
All traffic was legitimate — normal TTLs, clean handshake, expected redirects. No indicators of compromise.

📖 Full write-up: [session-1/wireshark-session1-writeup.md](session-1/wireshark-session1-writeup.md)

---

## 🔐 Session 2 — TLS/HTTPS Deep Dive & Certificate Analysis

### Capture
- Launched Wireshark on Windows with Edge browser
- Visited google.com, github.com over HTTPS
- Captured **5,538 packets** — 1,005 TLS packets identified

### Analyze
- **TLS Handshake:** Found Client Hello (packet 159) and Server Hello (packet 162)
- **Certificate Extraction:** Extracted Google certificate from browser (TLS 1.3 encrypts it in Wireshark)
- **Certificate Validation:** Verified issuer (Google Trust Services), validity dates, SHA-256 fingerprint
- **Cipher Suite:** Identified TLS_AES_256_GCM_SHA384 — military-grade encryption ✅
- **TCP Stream:** Used Follow TCP Stream — confirmed encrypted gibberish = encryption working correctly

### Key Finding
All HTTPS traffic was legitimate — valid certificate, strong cipher, encrypted data confirmed. No MITM indicators.

📖 Full write-up: [session-2/wireshark-session2-writeup.md](session-2/wireshark-session2-writeup.md)  
📋 Reference guide: [session-2/TLS-Certificate-Inspection-SOC-Guide.md](session-2/TLS-Certificate-Inspection-SOC-Guide.md)

---

## 💡 What This Proves

- ✅ **I can capture live traffic** — Not just read about it, but actually see it on my machine
- ✅ **I understand protocols** — DNS, HTTP, TCP, TLS handshakes in real packets
- ✅ **I think like a SOC analyst** — Certificate validation, cipher strength, MITM detection
- ✅ **I can use Wireshark filters** — `dns`, `http`, `tls`, `tcp.flags.syn==1` applied correctly
- ✅ **I document my work** — Full write-ups with field explanations + threat detection perspective

---

## 🛠️ Wireshark Filters Cheatsheet

```bash
# Protocol filtering
dns                          # DNS packets only
http                         # HTTP packets only
tcp                          # TCP packets only
tls                          # TLS/HTTPS packets only

# Find connection starts
tcp.flags.syn==1             # SYN packets
tcp.flags.syn==1 AND tcp.dstport==443  # HTTPS attempts

# TLS specific
tls.handshake.type==1        # Client Hello only
tls.handshake.type==2        # Server Hello only

# Filter by IP
ip.src==192.168.1.100        # From this IP
ip.dst==8.8.8.8              # To this IP

# Filter by port
tcp.port==443                # HTTPS
tcp.port==22                 # SSH
tcp.dstport==80              # HTTP incoming
```

---

## ⚠️ Red Flags (What to Look For in SOC Work)

| Protocol | Red Flag | Why It Matters |
|----------|----------|----------------|
| DNS | TTL < 60 seconds | Malware rotates IPs rapidly |
| DNS | Query to random-string domain | Typosquatting, phishing |
| HTTP | 3+ consecutive redirects | Phishing chain, malware delivery |
| HTTP | GET to unusual path (e.g., /admin.php?cmd=) | Possible exploitation attempt |
| TCP | SYN to unusual port (31337, 6666) | Backdoor activity |
| TCP | Many SYN with no SYN,ACK | Port scan or DoS |
| TLS | Self-signed certificate | No legitimate CA — likely attacker |
| TLS | Expired certificate | Misconfigured or compromised server |
| TLS | Subject mismatch | MITM attack in progress |
| TLS | Weak cipher (RC4, DES, MD5) | Vulnerable to cryptographic attacks |

---

## 🔥 For Job Interviews

**When asked "How would you analyze a PCAP file?"**

> "I start with Protocol Hierarchy (Statistics → Hierarchy) to see what protocols are present. For HTTP traffic I trace redirect chains and check for unusual paths. For DNS I look for fast-flux indicators like TTL under 60 seconds. For HTTPS, I filter by TLS and find the Client Hello to check the SNI — what domain the client is requesting. Then I find the Server Hello to check cipher suite strength. I extract the certificate via browser or openssl and verify: subject matches domain, issuer is a trusted CA, certificate is currently valid, fingerprint matches known-good records. I use Follow TCP Stream to confirm encryption is working — gibberish text is actually good news. Here's a real example from my Session 2 capture where I analyzed live Google HTTPS traffic and validated the certificate."

---

## 📊 Session Results Summary

| Session | Packets Captured | Protocols | Verdict |
|---------|-----------------|-----------|---------|
| Session 1 | 6,688 | DNS, HTTP, TCP | Legitimate — no IoCs |
| Session 2 | 5,538 | TLS, HTTPS, TCP | Legitimate — valid certificate, strong cipher |

---

## 🔄 Next Steps

- **Session 3:** Attack traffic (ARP spoofing, DNS tunneling, SYN floods)
- **Session 4:** CTF-style challenge (given malicious PCAP, extract IoCs)
- **Session 5:** Build cheatsheet + portfolio summary

---

## 📚 References

- [Wireshark Official Guide](https://www.wireshark.org/docs/)
- My learning path: Cisco Intro to Cybersecurity → TCM Linux 100 → TryHackMe labs

---

**Kishore Yuvaraj**
