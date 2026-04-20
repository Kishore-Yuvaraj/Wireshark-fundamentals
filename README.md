# Wireshark Fundamentals — Packet Analysis Mastery

A hands-on learning series where I capture real network traffic, analyze live protocols (DNS, HTTP, TCP), and document findings like a SOC analyst.

---

## 🎯 Quick Overview

**Goal:** Master Wireshark packet analysis from real network captures — moving from theory to practical SOC analyst skills.

**Current Status:**
- ✅ Session 1 — Live traffic capture & protocol fundamentals (COMPLETE)
- ⏳ Session 2–5 — Upcoming (TLS, attack traffic, CTF challenge, portfolio)

---

## 📁 What's in This Repo

```
wireshark-fundamentals/
├── README.md (this file)
└── session-1/
    ├── wireshark-session1-writeup.md ← READ THIS FIRST
    ├── session1-dns-http-capture.pcap (6,688 real packets)
    └── screenshots/
        ├── tcp-handshake-packets.png
        ├── dns-response.png
        └── http-redirect-flow.png
```

---

## 🚀 Session 1 — What I Did

### Capture
- Launched Wireshark on Windows
- Visited google.com, github.com, youtube.com in browser
- Captured 6,688 packets in ~30 seconds

### Analyze
- **DNS:** Filtered by `dns` → Found query for dns.google → Response: 8.8.4.4, 8.8.8.8
- **HTTP:** Filtered by `http` → Traced GET /online → 301 redirect → 200 OK response
- **TCP:** Filtered by `tcp.flags.syn==1` → Found 3-packet handshake (SYN → SYN,ACK → ACK)

### Key Finding
All traffic was **legitimate** — normal TTLs, clean handshake, expected redirects. No indicators of compromise.

---

## 💡 What This Proves

✅ **I can capture live traffic** — Not just read about it, but actually see it on my machine  
✅ **I understand protocols** — Can identify DNS queries, HTTP redirects, TCP sequences in real packets  
✅ **I think like a SOC analyst** — Documented threat indicators (TTL < 60s = suspicious)  
✅ **I can use Wireshark filters** — `dns`, `http`, `tcp.flags.syn==1` applied correctly  
✅ **I document my work** — Full write-up with field explanations + threat detection perspective  

---

## 🔥 For Job Interviews

**When asked "How would you analyze a PCAP file?"**

Show this repository and say:

> *"I start with Protocol Hierarchy (Statistics → Hierarchy) to see what's present. Then I filter by suspicious protocol — like `dns` to check for fast-flux malware indicators (TTL < 60s), or `http` to trace redirect chains. I drill into individual packets, read the dissection view to extract fields like source/destination IP, port, status codes. I look for indicators of compromise — unusual ports, suspicious domains, redirect chains. Here's a real example from my Session 1 capture: I found a normal 301 redirect, legitimate DNS resolver (Google's 8.8.8.8), and clean TCP handshake. No red flags."*

Then open your `session1-dns-http-capture.pcap` in Wireshark live and apply filters to show:
1. DNS query
2. HTTP GET request
3. TCP SYN packet

**This demonstrates:**
- Wireshark competency ✅
- Real experience (not theory) ✅
- Ability to explain what you see ✅
- Understanding of threat indicators ✅

---

## 📖 Session 1 Full Write-Up

For detailed analysis, read: **[session-1/wireshark-session1-writeup.md](./session-1/wireshark-session1-writeup.md)**

Includes:
- Complete packet breakdown (DNS, HTTP, TCP)
- Field explanations (TTL, Sequence numbers, Status codes)
- SOC threat detection perspective
- Why each protocol matters for security

---

## 🛠️ Wireshark Filters Cheatsheet

```
# Protocol filtering
dns                          # DNS packets only
http                         # HTTP packets only  
tcp                          # TCP packets only

# Find connection starts
tcp.flags.syn==1             # SYN packets
tcp.flags.syn==1 AND tcp.dstport==443  # HTTPS attempts

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
| **DNS** | TTL < 60 seconds | Malware rotates IPs rapidly |
| **DNS** | Query to random-string domain | Typosquatting, phishing |
| **HTTP** | 3+ consecutive redirects | Phishing chain, malware delivery |
| **HTTP** | GET to unusual path (e.g., `/admin.php?cmd=`) | Possible exploitation attempt |
| **TCP** | SYN to unusual port (31337, 6666) | Backdoor activity |
| **TCP** | Many SYN with no SYN,ACK | Port scan or DoS |

---

## 📊 Session 1 Results

**Capture details:**
- Total packets: 6,688
- Duration: ~30 seconds
- Protocols: DNS, HTTP, TCP, TLS
- Verdict: **Legitimate traffic — no IoCs detected**

**Key stats:**
- DNS queries: 4 (normal for 3 website visits)
- HTTP packets: 6 (request + redirect + favicon)
- TCP handshakes: 3 packets (SYN, SYN,ACK, ACK)

---

## 🎯 Interview Talking Points

**Use this story in interviews:**

*"I built a hands-on Wireshark learning series to bridge the gap between networking theory and practical SOC analysis. In Session 1, I captured real traffic from my machine, filtered it by protocol, and analyzed individual packets. I traced a complete HTTP conversation including a 301 redirect, identified a TCP 3-way handshake with legitimate sequence numbers, and verified DNS responses from Google's public resolver. I documented threat indicators — like TTL values (normal ones are 300+), redirect chains (legitimate sites have ≤1), and port numbers (unusual ones like 31337 = backdoor). This taught me not just how to use Wireshark, but how to think like a SOC analyst when reading PCAP files."*

**Then show:**
1. Open your GitHub repo
2. Click `session-1/wireshark-session1-writeup.md` (show they can read full analysis)
3. Download the `.pcap` file and open in Wireshark
4. Apply `dns` filter and explain what you see
5. Reference the threat indicators table

---

## 🔄 Next Steps

- **Session 2:** TLS/HTTPS handshake analysis (why encrypted traffic still reveals patterns)
- **Session 3:** Attack traffic (ARP spoofing, DNS tunneling, SYN floods)
- **Session 4:** CTF-style challenge (given malicious PCAP, extract IoCs)
- **Session 5:** Build cheatsheet + portfolio summary

---

## 📚 References

- [Wireshark Official Guide](https://www.wireshark.org/docs/)
- My learning path: Cisco Intro to Cybersecurity → TCM Linux 100 → TryHackMe labs

---

**Kishore Yuvaraj**
