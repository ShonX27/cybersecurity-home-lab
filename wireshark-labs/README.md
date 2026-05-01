# Wireshark Network Traffic Analysis

## Objective
Capture and analyze live network traffic using Wireshark to identify normal vs suspicious activity, practice reading packet data, and understand common protocols used in SOC environments.

---

## Tools Used
- Wireshark 4.x
- Kali Linux VM
- Windows 10 VM (traffic generator)
- VirtualBox Internal Network

---

## Lab 1 — Capturing Basic Traffic

### Steps
1. Open Wireshark on Kali Linux
2. Select the network interface connected to `labnet` (e.g., `eth0`)
3. Click the blue shark fin to start capturing
4. On the Windows VM, open a browser and visit `http://example.com`
5. Stop the capture after 30 seconds (red square button)
6. Save the capture file as `basic-traffic.pcap`

### What to Look For
- DNS queries (port 53) — the browser asking "what's the IP for example.com?"
- TCP three-way handshake (SYN, SYN-ACK, ACK)
- HTTP GET requests (port 80)

### Filters Used
```
dns                         # show only DNS traffic
http                        # show only HTTP traffic
tcp.flags.syn == 1          # show TCP SYN packets (new connections)
ip.addr == 192.168.56.30    # filter by Windows VM IP
```

### Key Finding
Observed a DNS query for `example.com` followed immediately by a TCP connection to the resolved IP on port 80. The HTTP GET request was visible in plaintext — demonstrating why HTTPS is important.

---

## Lab 2 — Detecting a Port Scan

### Steps
1. Start a Wireshark capture on Kali
2. From Kali terminal, run an Nmap scan against the Windows VM:
```bash
nmap -sS 192.168.56.30
```
3. Stop the capture and analyze

### Filters Used
```
tcp.flags.syn == 1 && tcp.flags.ack == 0    # SYN packets only (scan traffic)
ip.src == 192.168.56.10                      # traffic from Kali (attacker)
```

### What a Port Scan Looks Like
- A single source IP sending SYN packets to many different destination ports rapidly
- RST/ACK responses from closed ports
- SYN-ACK responses from open ports

### Key Finding
Nmap scan generated over 1,000 SYN packets in under 2 seconds to the target IP. This pattern — one source, many destination ports, rapid timing — is a clear indicator of reconnaissance activity. In a real SOC, this would trigger a SIEM alert.

---

## Lab 3 — Analyzing Suspicious DNS Traffic

### Steps
1. Generate DNS queries from Windows VM to an external domain
2. Capture traffic and filter by `dns`
3. Look for unusually long domain names or high-frequency queries

### Filters Used
```
dns.qry.name contains "suspicious"
dns && ip.dst != 192.168.56.1      # DNS going somewhere other than local resolver
```

### Key Finding
Normal DNS queries are short and infrequent. DNS tunneling or C2 beaconing often shows up as very long subdomains or regular query intervals every X seconds — a pattern no legitimate user browsing would generate.

---

## Summary of IOCs Practiced

| Indicator | Protocol | What It Means |
|-----------|----------|---------------|
| Rapid SYN to many ports | TCP | Port scan / reconnaissance |
| DNS query for unknown domain | DNS | Possible C2 or phishing domain |
| HTTP in cleartext | HTTP | Credential exposure risk |
| Large data transfer outbound | TCP | Possible data exfiltration |

---

## What I Learned
- How to read and interpret packet captures in Wireshark
- How to write display filters to isolate relevant traffic
- What a port scan looks like at the packet level — critical knowledge for a SOC analyst triaging SIEM alerts
- The difference between a normal TCP handshake and anomalous connection patterns

---

## Screenshots
*(Add screenshots of: Wireshark capturing traffic, the SYN scan filter applied, and a DNS query expanded to show its fields)*
