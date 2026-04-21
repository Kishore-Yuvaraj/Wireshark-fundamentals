# TLS Certificate Inspection in Wireshark — SOC Analyst Deep Dive

## Part 1: Why Inspect TLS Certificates Even When Connection Succeeds?

### The Critical Truth

**A successful TLS handshake does NOT mean the connection is legitimate.**

Even if your browser shows a lock icon 🔒 and the connection is encrypted, you could be connected to:
- A **phishing server** with a valid certificate (criminals buy them)
- A **compromised server** that was hacked (certificate is real but server isn't)
- A **MITM attacker** who intercepted your traffic (certificate validation failed but you didn't notice)

### Real-World Example

**Incident:** Company employee received email claiming to be from IT: "Update your password here: https://company-secure-login.com"

- ✅ Browser shows lock icon (TLS certificate exists)
- ✅ Connection encrypted (TLS handshake succeeded)
- ❌ **But:** Certificate was issued to attacker's domain, not company's real domain
- **Result:** Employee entered credentials, attacker logged into company systems

**The analyst's job:** Inspect that certificate and catch the mismatch BEFORE the employee visits.

---

## Part 2: TLS 1.3 Certificate Validation Techniques

When TLS 1.3 encrypts the certificate in the handshake, analysts use these methods:

### Technique 1: Browser Certificate Extraction (Quickest)

**How:**
```
1. Visit the HTTPS site in a browser
2. Click lock icon → Connection secure → Certificate details
3. Export/view the certificate
4. Check issuer, subject, validity dates, fingerprint
```

**Pros:**
- Fast (30 seconds)
- No tools needed
- Immediate verification

**Cons:**
- Only works if site is still online
- Doesn't work for internal servers/IPs

**Use case:** Verifying live websites during incident response

---

### Technique 2: OpenSSL Command-Line

**How:**
```bash
# Get certificate from server
openssl s_client -connect google.com:443 -showcerts < /dev/null

# Extract and verify
openssl x509 -in certificate.pem -text -noout

# Check expiration
openssl x509 -in certificate.pem -noout -dates

# Verify against CA bundle
openssl verify -CAfile ca-bundle.crt certificate.pem
```

**Output:**
```
Subject: CN=*.google.com
Issuer: CN=Google Trust Services
Not Before: Mar 30 2026
Not After: Jun 22 2026
SHA256 Fingerprint: 210f478de4275dd6283abd2a8fcca589d1b78c89ec9ad7a414056a3cc932dc74
```

**Pros:**
- Works with IP addresses + domains
- Scriptable (automate validation)
- Shows full certificate chain
- Extract specific fields

**Cons:**
- Requires command-line access
- Requires openssl installed

**Use case:** Automated validation of multiple servers, server-side analysis

---

### Technique 3: SSLSCAN / Qualys SSL Labs

**How:**
```bash
# Using sslscan
sslscan --no-failed google.com

# Or visit online:
https://www.ssllabs.com/ssltest/analyze.html?d=google.com
```

**Output:**
```
Certificate Information:
  Subject: *.google.com
  Issuer: Google Trust Services
  Valid from: Mar 30, 2026
  Valid until: Jun 22, 2026
  Public Key: RSA 2048 bits
  Signature Algorithm: sha256WithRSAEncryption

Cipher Suites:
  TLS 1.3: TLS_AES_256_GCM_SHA384
  TLS 1.3: TLS_CHACHA20_POLY1305_SHA256
  TLS 1.2: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384

Security Grade: A+
```

**Pros:**
- Comprehensive security assessment
- Shows all cipher suites
- Detects SSL/TLS vulnerabilities
- Online version (no installation)

**Cons:**
- Slower than CLI
- Online version logs your queries
- May not work for internal IPs

**Use case:** Complete security posture analysis, vulnerability assessment

---

### Technique 4: Certificate Transparency Logs

**How:**
```
1. Visit https://crt.sh
2. Search for domain name
3. View all certificates issued for that domain
4. Check dates, CAs, fingerprints
```

**What you learn:**
```
Certificate Search Results:
  Certificate ID: 1234567890
  Domain: google.com, *.google.com
  Issuer: Google Trust Services
  Issued: 2026-03-30
  Expires: 2026-06-22
  Fingerprint: 210f478de4275dd6283abd2a8fcca589d1b78c89ec9ad7a414056a3cc932dc74
  
  [Previous certificates for same domain...]
  Issued: 2025-12-15 (old certificate)
  Issued: 2025-09-10 (even older)
```

**Pros:**
- See ALL certificates issued for a domain (historical)
- Detect certificate replacement attacks
- Public record (no authentication needed)
- Shows certificate lifecycle

**Cons:**
- Only shows public CAs (not internal corporate certs)
- Data is public (privacy concern for sensitive domains)

**Use case:** Historical analysis, detecting compromise timeline

---

### Technique 5: PCAP Analysis with TLS Decryption (Advanced)

If you have:
- Access to private keys
- SSL/TLS session keys
- Wireshark configured for decryption

**How in Wireshark:**
```
Edit → Preferences → Protocols → TLS → 
[+] Add SSLKEYLOGFILE from browser environment
```

This decrypts recorded HTTPS traffic for inspection.

**Pros:**
- See actual HTTP requests/responses inside HTTPS
- Inspect payload data

**Cons:**
- Requires private keys (rarely available)
- Privacy/legal concerns
- Complex setup

**Use case:** Lab environments, authorized incident response

---

## Part 3: Suspicious Certificate Scenario — Incident Response Procedure

### Red Flags Checklist

When you discover a certificate with red flags:

| Red Flag | Severity | What It Means |
|----------|----------|---------------|
| **Self-signed cert** | 🔴 CRITICAL | No legitimate CA backing it; likely attacker |
| **Expired cert** | 🔴 CRITICAL | Server misconfigured OR hijacked (attacker didn't renew) |
| **Subject mismatch** | 🔴 CRITICAL | Visiting google.com but cert is for evil.com = MITM |
| **Unknown/suspicious issuer** | 🔴 CRITICAL | Not a recognized CA; possibly forged |
| **Weak signature (SHA1, MD5)** | 🟡 HIGH | Old weak hash; vulnerable to forgery |
| **Unusual certificate chain** | 🟡 HIGH | Extra CAs not seen before; possible hijacking |
| **Recently issued** | 🟡 MEDIUM | Cert issued TODAY for old domain; possible takeover |
| **Mismatched IP/DNS** | 🟡 MEDIUM | IP from China, domain says USA; possible hijacking |

---

### Incident Response Workflow

**STEP 1: Isolate & Contain**

```
1. DO NOT TRUST THE CONNECTION
2. Block the IP at firewall immediately
3. Alert security team with:
   - IP address
   - Domain name (SNI)
   - Certificate details (subject, issuer, fingerprint)
   - Timestamp when seen
   - Which user/machine accessed it
```

**Example alert:**
```
CRITICAL: Suspicious HTTPS Certificate Detected

Domain (SNI): company-secure-login.com
IP Address: 10.10.10.50 (known-bad IP range)
Certificate Issuer: Self-Signed
Certificate Subject: company-secure-login.com
Valid From: April 21, 2026 (TODAY — newly issued)
Valid Until: April 21, 2027

Threat Assessment: PHISHING / MALWARE
Action: Block IP immediately
Escalate to incident response team
```

---

**STEP 2: Identify Affected Systems**

```bash
# In Wireshark or logs, find all connections to that IP
Filter: ip.dst == 10.10.10.50

# Questions to answer:
- How many machines connected?
- How much data was transferred?
- Which user IDs were involved?
- Did credentials get transmitted? (look for POST requests)
```

---

**STEP 3: Investigate the Source**

```
1. Reverse-lookup the IP address:
   - whois 10.10.10.50
   - ip-location tools
   
2. Check if it's known malicious:
   - AbuseIPDB.com
   - Shodan.io
   - Threat intelligence feeds

3. Check certificate fingerprint:
   - Search VirusTotal for fingerprint
   - Check Certificate Transparency logs
   - See if other victims recorded it

4. Timeline analysis:
   - When was certificate issued?
   - When was this IP first seen?
   - Any pattern to when it was accessed?
```

---

**STEP 4: Determine Attack Vector**

```
Ask:
- Is this a phishing email campaign?
  (Check email headers for sender spoofing)
  
- Is this a DNS hijacking?
  (Look at DNS queries: did user query company.com and get 10.10.10.50?)
  
- Is this MITM on compromised network?
  (Check if legitimate DNS resolved to wrong IP)
  
- Is this credential harvester?
  (Did POST request contain username/password fields?)
```

---

**STEP 5: Forensics & Evidence Collection**

```
Preserve:
1. PCAP file of the suspicious connection
2. Screenshot of certificate details
3. Firewall logs showing access
4. Endpoint logs (browser history, process execution)
5. Email if this was phishing (headers, body, links)
6. DNS query logs (what domain did user type vs. what IP resolved)
```

---

**STEP 6: Incident Report**

```
Title: Detected phishing certificate for company-secure-login.com

Summary:
  On April 21, 2026 at 14:35 UTC, user john.smith@company.com 
  attempted to access https://company-secure-login.com 
  which resolved to IP 10.10.10.50. 
  Wireshark analysis revealed self-signed certificate with 
  suspicious issuer "Self-Signed". 
  Connection was blocked at firewall.

Indicators of Compromise (IoCs):
  - IP: 10.10.10.50
  - Domain: company-secure-login.com
  - Certificate Fingerprint: [SHA256 hash]
  - Cipher: TLS_RSA_WITH_RC4_128_SHA (WEAK)

Impact:
  - 1 user attempted connection
  - No credentials believed compromised
  - No data loss

Remediation:
  - IP blocked company-wide
  - User trained on phishing detection
  - DNS monitoring enabled for company-*.com domains
  - Email security rules updated

Status: RESOLVED
```

---

## Part 4: Decision-Making Framework — Legitimate vs. Malicious

When TLS handshake appears normal but certificate is suspicious:

### Decision Tree

```
START: Found suspicious TLS certificate

1. Is the certificate CURRENTLY VALID?
   ├─ NO (expired)  → 🚩 MALICIOUS (or misconfigured)
   │                  Action: Block IP, alert team
   │
   └─ YES → Continue to 2

2. Does SUBJECT MATCH the domain you're visiting?
   ├─ NO (mismatch)  → 🚩 MALICIOUS (MITM)
   │                   Action: Block IP, block domain in DNS
   │
   └─ YES → Continue to 3

3. Is the ISSUER a known, legitimate CA?
   ├─ NO (unknown/self-signed) → 🚩 MALICIOUS
   │                              Action: Block immediately
   │
   └─ YES → Continue to 4

4. Does the FINGERPRINT match known-good records?
   ├─ NO (fingerprint changed) → 🚩 MALICIOUS (certificate hijacking)
   │                              Action: Investigate compromise
   │
   └─ YES → Continue to 5

5. Check IP GEOLOCATION & REPUTATION
   ├─ IP from high-risk country → 🟡 INVESTIGATE
   ├─ IP in known-bad list (AbuseIPDB) → 🚩 MALICIOUS
   │
   └─ IP looks normal → Continue to 6

6. Analyze TRAFFIC PATTERN
   ├─ Large data transfer (credentials?) → 🟡 INVESTIGATE
   ├─ Unusual ports → 🟡 INVESTIGATE
   └─ Normal HTTPS usage → ✅ LIKELY LEGITIMATE

7. Check CERTIFICATE CHAIN
   ├─ Missing intermediate certs → 🟡 INVESTIGATE
   └─ Complete chain from root CA → ✅ LIKELY LEGITIMATE

DECISION: 
  ✅ All checks pass → LEGITIMATE (but monitor)
  🟡 Some concerns → INVESTIGATE FURTHER
  🚩 Red flags found → MALICIOUS (block & remediate)
```

---

### Additional Investigation Tools

When decision tree is inconclusive:

| Tool | Purpose | Command |
|------|---------|---------|
| **VirusTotal** | Check if certificate fingerprint is known malicious | virustotal.com (search fingerprint) |
| **URLhaus** | Check if domain is phishing/malware | urlhaus-api.abuse.ch |
| **Shodan.io** | See what services are running on the IP | shodan search 10.10.10.50 |
| **AbuseIPDB** | Check if IP has abuse history | abuseipdb.com |
| **whois** | Get IP ownership info | whois 10.10.10.50 |
| **geoiplookup** | Get IP geographic location | geoiplookup 10.10.10.50 |
| **nslookup history** | Check past DNS resolutions | dnsdumpster.com |

---

## Part 5: Practical SOC Analyst Workflow

### Scenario: You See Suspicious TLS Traffic in PCAP

**Day 1 — Initial Detection (30 minutes)**

```
1. Open PCAP in Wireshark
2. Filter: tls
3. Find suspicious connections:
   - Unusual ports (not 443)
   - Unknown IPs
   - Many failed handshakes
4. Right-click → Follow TCP Stream
5. Check if data looks like HTTPS or random gibberish
6. Document IP, port, timing in incident log
```

**Day 1 — Quick Validation (30 minutes)**

```
1. Extract certificate using openssl
2. Check: Issuer, Subject, Validity, Fingerprint
3. Compare against known-good list
4. Run through decision tree
5. If suspicious: escalate immediately
```

**Day 2 — Detailed Investigation (2-4 hours)**

```
1. Check firewall/proxy logs for that IP
2. Find all machines that connected to it
3. Check if credentials were exfiltrated:
   - Look for POST requests (user auth)
   - Check if form fields exist in HTTP payloads
4. Search threat intel feeds for IP/cert
5. Contact network team: is this authorized?
```

**Day 3 — Forensics & Remediation (ongoing)**

```
1. If malicious:
   - Block IP globally
   - Update IDS/IPS rules
   - Scan machines that accessed it
   - Reset affected user passwords
   - Notify users of compromised accounts

2. If legitimate:
   - Document certificate details
   - Add to whitelist
   - Monitor for changes
   - Update firewall allowlist
```

---

## Part 6: Interview Answers Using This Knowledge

### Interview Question 1: "Why inspect TLS certificates in secure connections?"

**Answer:**
> "A successful TLS handshake and browser lock icon don't guarantee legitimacy. Attackers can obtain valid certificates for phishing domains or compromise legitimate servers. As a SOC analyst, I inspect certificates to verify: 1) the issuer is a known CA, 2) the subject matches the domain being visited, 3) the certificate is currently valid, 4) the fingerprint matches known-good records from Certificate Transparency logs. I once saw a case where an employee was tricked by a phishing email linking to 'company-secure-login.com' with a valid certificate issued to an attacker's domain. Inspecting the certificate revealed the subject mismatch, which would have prevented credential theft."

---

### Interview Question 2: "How do you validate TLS certificates in TLS 1.3?"

**Answer:**
> "TLS 1.3 encrypts the certificate in the handshake, so Wireshark can't display it directly. I use alternative methods: First, I extract the certificate using the browser's lock icon feature or openssl s_client command. Then I verify it against Certificate Transparency logs and check the fingerprint against known-good records. For automated analysis, I use tools like sslscan or Qualys SSL Labs to get comprehensive certificate and cipher information. I can also check the Subject Name Indication (SNI) field in the Client Hello packet to see what domain the client is requesting, even if I can't see the full certificate."

---

### Interview Question 3: "You find a suspicious TLS certificate. What do you do?"

**Answer:**
> "I follow incident response protocol. First, I isolate and contain by blocking the IP immediately at the firewall. Then I identify the scope: which machines connected, how much data transferred, which users were involved. I investigate the source using whois, AbuseIPDB, and Shodan to understand the IP. I check Certificate Transparency logs to see if other victims recorded this certificate. I determine the attack vector: is it phishing, DNS hijacking, or MITM? I preserve all evidence: PCAP file, certificate screenshots, firewall logs, DNS queries, email headers if applicable. Finally, I write an incident report and remediate: block globally, update IDS rules, reset credentials, notify affected users. The goal is not just to stop this incident but to prevent it from happening again."

---

## Summary Table: Validation Techniques Quick Reference

| Technique | Speed | Scope | Tools Needed | Best For |
|-----------|-------|-------|--------------|----------|
| Browser extraction | ⚡ 30 sec | Single domain | Browser | Quick verification |
| OpenSSL CLI | ⚡ 1 min | Domain + IP | openssl CLI | Scripting, servers |
| SSLSCAN | ⚡ 2 min | Single domain | sslscan | Security posture |
| Qualys SSL Labs | 🟡 5 min | Single domain | Web browser | Comprehensive audit |
| Cert Transparency logs | 🟡 2 min | Historical | Web browser | Timeline analysis |
| PCAP + TLS decryption | 🔴 1 hour | Encrypted data | Wireshark + keys | Full payload inspection |

---
*Kishore Yuvaraj | April 21, 2026*
