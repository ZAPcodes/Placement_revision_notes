# 12 — Practice MCQs + Numericals + Cheatsheet (read before OA / interview)

## Part A — Practice MCQs

Answers are hidden under each question.

### Models & devices
**1.** Which OSI layer is responsible for process-to-process delivery?
(a) Network (b) Transport (c) Session (d) Data Link
<details><summary>Answer</summary>(b) — via port numbers.</details>

**2.** The PDU at the Data Link layer is called:
(a) Packet (b) Segment (c) Frame (d) Bit
<details><summary>Answer</summary>(c)</details>

**3.** Encryption and compression belong to which OSI layer?
(a) Application (b) Presentation (c) Session (d) Transport
<details><summary>Answer</summary>(b)</details>

**4.** A router operates at which layer?
(a) Physical (b) Data Link (c) Network (d) Transport
<details><summary>Answer</summary>(c)</details>

**5.** A 24-port switch (no VLANs) has how many collision and broadcast domains?
(a) 1, 1 (b) 24, 1 (c) 24, 24 (d) 1, 24
<details><summary>Answer</summary>(b)</details>

**6.** Which device breaks up broadcast domains?
(a) Hub (b) Switch (c) Bridge (d) Router
<details><summary>Answer</summary>(d) — VLANs on a switch also do.</details>

**7.** As a packet crosses routers, which stays the same end to end (no NAT)?
(a) Source MAC (b) Destination MAC (c) Source and destination IP (d) TTL
<details><summary>Answer</summary>(c)</details>

### Data link
**8.** Which protocol maps an IP address to a MAC address?
(a) RARP (b) ARP (c) DHCP (d) DNS
<details><summary>Answer</summary>(b)</details>

**9.** An ARP request is sent as ___ and the reply as ___.
(a) Unicast, broadcast (b) Broadcast, unicast (c) Multicast, unicast (d) Broadcast, broadcast
<details><summary>Answer</summary>(b)</details>

**10.** Maximum throughput of slotted ALOHA is:
(a) 18.4% (b) 36.8% (c) 50% (d) 100%
<details><summary>Answer</summary>(b). Pure ALOHA is 18.4%.</details>

**11.** Wi-Fi uses CSMA/CA instead of CSMA/CD mainly because:
(a) CA is faster (b) Wireless stations can't reliably detect collisions while transmitting (c) Wi-Fi has no collisions (d) CD needs fibre
<details><summary>Answer</summary>(b)</details>

**12.** With a 3-bit sequence number, the maximum sender window for Go-Back-N and Selective Repeat is:
(a) 8, 8 (b) 7, 4 (c) 7, 7 (d) 4, 4
<details><summary>Answer</summary>(b) — 2^k − 1 and 2^(k−1).</details>

**13.** Single-bit parity can detect:
(a) All errors (b) All odd numbers of bit errors (c) All even numbers of bit errors (d) Only burst errors
<details><summary>Answer</summary>(b)</details>

**14.** To correct up to d bit errors, the minimum Hamming distance needed is:
(a) d (b) d + 1 (c) 2d (d) 2d + 1
<details><summary>Answer</summary>(d)</details>

**15.** Minimum Ethernet frame size is:
(a) 46 bytes (b) 64 bytes (c) 512 bytes (d) 1500 bytes
<details><summary>Answer</summary>(b). 46 is the minimum payload; 1500 is the MTU.</details>

### Network layer
**16.** `172.20.10.5` belongs to:
(a) Class A public (b) Class B private range (c) Class C private (d) Class D
<details><summary>Answer</summary>(b) — 172.16.0.0/12 is private.</details>

**17.** A host with IP `169.254.12.7` most likely:
(a) Is a DNS server (b) Failed to get an address from DHCP (c) Is on a class C network (d) Is a multicast group member
<details><summary>Answer</summary>(b) — APIPA.</details>

**18.** Usable hosts in a /28 subnet:
(a) 16 (b) 14 (c) 30 (d) 12
<details><summary>Answer</summary>(b)</details>

**19.** The IPv4 fragment offset is measured in units of:
(a) 1 byte (b) 4 bytes (c) 8 bytes (d) 16 bytes
<details><summary>Answer</summary>(c)</details>

**20.** Where are IPv4 fragments reassembled?
(a) Next router (b) Every router (c) Destination host (d) Source host
<details><summary>Answer</summary>(c)</details>

**21.** Which is **not** true about IPv6?
(a) 128-bit addresses (b) Routers don't fragment (c) Has a header checksum (d) No broadcast
<details><summary>Answer</summary>(c)</details>

**22.** Count-to-infinity is a problem in:
(a) Link state routing (b) Distance vector routing (c) Path vector (d) Static routing
<details><summary>Answer</summary>(b)</details>

**23.** OSPF uses which algorithm?
(a) Bellman-Ford (b) Dijkstra (c) Kruskal (d) Floyd-Warshall
<details><summary>Answer</summary>(b)</details>

**24.** BGP runs over:
(a) UDP 520 (b) TCP 179 (c) IP directly (d) UDP 53
<details><summary>Answer</summary>(b). RIP = UDP 520, OSPF = IP protocol 89.</details>

**25.** Traceroute relies on:
(a) ARP replies (b) ICMP Time Exceeded messages as TTL expires (c) DNS lookups (d) TCP RST
<details><summary>Answer</summary>(b)</details>

**26.** A router has routes `10.0.0.0/8`, `10.1.0.0/16`, `10.1.2.0/24`. Packet to `10.1.2.9` uses:
(a) /8 (b) /16 (c) /24 (d) Default route
<details><summary>Answer</summary>(c) — longest prefix match.</details>

### Transport layer
**27.** UDP header size is:
(a) 8 bytes (b) 20 bytes (c) 16 bytes (d) 40 bytes
<details><summary>Answer</summary>(a)</details>

**28.** Which consumes a TCP sequence number despite carrying no data?
(a) ACK (b) SYN (c) RST (d) PSH
<details><summary>Answer</summary>(b) — FIN does too.</details>

**29.** TIME_WAIT state is entered by:
(a) The server always (b) The side that closes first (active close) (c) Both sides (d) The side that receives the first FIN
<details><summary>Answer</summary>(b)</details>

**30.** In TCP Reno, after 3 duplicate ACKs, cwnd becomes:
(a) 1 MSS (b) Half the previous cwnd (fast recovery) (c) Unchanged (d) Double
<details><summary>Answer</summary>(b). Tahoe resets to 1.</details>

**31.** The receiver window (rwnd) is used for:
(a) Congestion control (b) Flow control (c) Error control (d) Routing
<details><summary>Answer</summary>(b)</details>

**32.** During slow start, cwnd grows:
(a) Linearly per RTT (b) Exponentially (doubles per RTT) (c) Stays constant (d) Halves
<details><summary>Answer</summary>(b)</details>

**33.** Which pair both use UDP?
(a) HTTP, FTP (b) DNS, DHCP (c) SMTP, SSH (d) Telnet, IMAP
<details><summary>Answer</summary>(b)</details>

### Application layer & security
**34.** Which DNS record maps a domain to its mail server?
(a) A (b) CNAME (c) MX (d) PTR
<details><summary>Answer</summary>(c)</details>

**35.** DNS uses TCP for:
(a) All queries (b) Zone transfers and large responses (c) Reverse lookups only (d) Never
<details><summary>Answer</summary>(b)</details>

**36.** Which HTTP method is **not** idempotent?
(a) GET (b) PUT (c) DELETE (d) POST
<details><summary>Answer</summary>(d)</details>

**37.** A logged-in user tries to access an admin page they don't have rights to. Correct status code:
(a) 400 (b) 401 (c) 403 (d) 404
<details><summary>Answer</summary>(c). 401 is for unauthenticated.</details>

**38.** HTTP/2's key improvement over HTTP/1.1 is:
(a) Uses UDP (b) Multiplexing multiple streams over one TCP connection (c) Plain-text headers (d) One connection per request
<details><summary>Answer</summary>(b). HTTP/3 is the one on UDP (QUIC).</details>

**39.** Which protocol keeps email on the server and syncs across devices?
(a) SMTP (b) POP3 (c) IMAP (d) FTP
<details><summary>Answer</summary>(c)</details>

**40.** FTP's control connection uses port:
(a) 20 (b) 21 (c) 22 (d) 23
<details><summary>Answer</summary>(b). 20 is data in active mode.</details>

**41.** DHCP message order:
(a) Request, Offer, Discover, Ack (b) Discover, Offer, Request, Ack (c) Offer, Discover, Ack, Request (d) Discover, Request, Offer, Ack
<details><summary>Answer</summary>(b) — DORA.</details>

**42.** TLS uses asymmetric cryptography mainly to:
(a) Encrypt all data (b) Authenticate the server and establish a shared symmetric key (c) Compress data (d) Replace TCP
<details><summary>Answer</summary>(b)</details>

**43.** RTS/CTS in Wi-Fi addresses the:
(a) Exposed terminal problem (b) Hidden terminal problem (c) Count-to-infinity problem (d) Silly window syndrome
<details><summary>Answer</summary>(b)</details>

**44.** Fake ARP replies used to intercept LAN traffic is:
(a) DNS poisoning (b) ARP spoofing (c) SYN flood (d) Phishing
<details><summary>Answer</summary>(b)</details>

**45.** Which operates at the network layer?
(a) TLS (b) IPSec (c) SSH (d) HTTPS
<details><summary>Answer</summary>(b)</details>

---

## Part B — Numerical drills

**N1. Delays.** 2 MB file, 10 Mbps link, 1000 km, signal speed 2×10⁸ m/s. Find Tt and Tp.
<details><summary>Answer</summary>Tt = 16×10⁶ bits / 10⁷ = <b>1.6 s</b>. Tp = 10⁶ / 2×10⁸ = <b>5 ms</b>. (Take 1 MB = 10⁶ bytes unless told otherwise.)</details>

**N2. Subnetting.** For `10.1.5.130/27`, find network, broadcast, usable host count.
<details><summary>Answer</summary>Block = 32 → network <b>10.1.5.128</b>, broadcast <b>10.1.5.159</b>, <b>30</b> hosts (.129–.158).</details>

**N3. Prefix choice.** Smallest subnet for 500 hosts?
<details><summary>Answer</summary><b>/23</b> → 2⁹ − 2 = 510 (a /24 gives only 254).</details>

**N4. CRC.** Data `100100`, generator `1101`. Find the transmitted frame.
<details><summary>Answer</summary>Append 3 zeros → `100100000` ÷ `1101` → remainder <b>001</b>. Frame: <b>100100001</b>.</details>

**N5. Hamming.** How many redundant bits for 11 data bits?
<details><summary>Answer</summary>2^r ≥ 11 + r + 1 → r = <b>4</b> (16 ≥ 16). Total 15 bits.</details>

**N6. Sliding window efficiency.** Tt = 2 ms, Tp = 10 ms, window = 5 (GBN). Efficiency?
<details><summary>Answer</summary>a = 5 → 1 + 2a = 11 → efficiency = 5/11 ≈ <b>45.5%</b>. For 100%, window ≥ 11.</details>

**N7. CSMA/CD.** 1 Gbps Ethernet, minimum frame 512 bytes, signal speed 2×10⁸ m/s. Max cable length?
<details><summary>Answer</summary>Tt = 4096 bits / 10⁹ = 4.096 µs. Need Tt ≥ 2Tp → Tp ≤ 2.048 µs → d ≤ 2.048 µs × 2×10⁸ = <b>409.6 m</b>.</details>

**N8. ALOHA.** A pure ALOHA channel is 200 kbps. Max useful throughput?
<details><summary>Answer</summary>0.184 × 200 = <b>≈ 36.8 kbps</b>. (Slotted: ≈ 73.6 kbps.)</details>

**N9. Fragmentation.** 2400-byte datagram (20-byte header), MTU 820. Fragments' data sizes, offsets, MF?
<details><summary>Answer</summary>Data per fragment = 800 (multiple of 8 ✓). Total data 2380 → <b>800, 800, 780</b>; offsets <b>0, 100, 200</b>; MF <b>1, 1, 0</b>.</details>

**N10. Congestion window.** ssthresh = 8, cwnd starts at 1 MSS. At cwnd = 12, three duplicate ACKs arrive. Trace for Reno and Tahoe.
<details><summary>Answer</summary>1 → 2 → 4 → 8 (ssthresh) → 9 → 10 → 11 → 12 → loss. New ssthresh = <b>6</b>.<br>Reno: cwnd = <b>6</b>, then 7, 8, ... (linear).<br>Tahoe: cwnd = <b>1</b>, then 2, 4, 6 (capped), 7, ...</details>

**N11. Shannon + Nyquist.** B = 1 MHz, SNR = 63. Max capacity and signal levels needed?
<details><summary>Answer</summary>C = 10⁶ × log₂(64) = <b>6 Mbps</b>. Nyquist: 6×10⁶ = 2 × 10⁶ × log₂L → log₂L = 3 → <b>L = 8</b>.</details>

**N12. Sequence number wrap-around.** How long until TCP's 32-bit sequence space wraps on a 1 Gbps link?
<details><summary>Answer</summary>2³² bytes ÷ (10⁹/8 bytes/s) ≈ <b>34.4 s</b>.</details>

**N13. Window-limited throughput.** 64 KB window, RTT 50 ms. Max throughput of one TCP connection?
<details><summary>Answer</summary>65,535 × 8 / 0.05 ≈ <b>10.5 Mbps</b>, regardless of link speed.</details>

**N14. Full mesh.** Links and ports per device for 8 devices?
<details><summary>Answer</summary>8 × 7 / 2 = <b>28</b> links; <b>7</b> ports each.</details>

---

## Part C — Cheatsheet

### OSI at a glance
| # | Layer | PDU | Address | Device | Key protocols |
|---|---|---|---|---|---|
| 7 | Application | Data | — | Gateway | HTTP, DNS, SMTP, FTP, DHCP, SSH |
| 6 | Presentation | Data | — | — | TLS*, JPEG, ASCII |
| 5 | Session | Data | — | — | RPC, NetBIOS |
| 4 | Transport | Segment / Datagram | Port | L4 firewall/LB | TCP, UDP |
| 3 | Network | Packet | IP | Router | IP, ICMP, OSPF, IPSec |
| 2 | Data Link | Frame | MAC | Switch, bridge | Ethernet, ARP, PPP, 802.11 |
| 1 | Physical | Bits | — | Hub, repeater | Cables, signals |

Node-to-node (L2) · host-to-host (L3) · process-to-process (L4).

### Formulas
| What | Formula |
|---|---|
| Transmission delay | Tt = L / B |
| Propagation delay | Tp = d / v |
| a | Tp / Tt |
| Stop-and-Wait efficiency | 1 / (1 + 2a) |
| GBN / SR efficiency | N / (1 + 2a), max 1 |
| Window for 100% | N ≥ 1 + 2a |
| GBN / SR max window | 2^k − 1 / 2^(k−1) |
| Pure / slotted ALOHA max | 18.4% / 36.8% |
| CSMA/CD min frame | L ≥ 2 × Tp × B |
| Nyquist | C = 2B log₂L |
| Shannon | C = B log₂(1 + SNR), SNR_dB = 10 log₁₀ SNR |
| Hamming redundant bits | 2^r ≥ m + r + 1 |
| Detect d / correct d errors | distance d+1 / 2d+1 |
| Hosts in /n | 2^(32−n) − 2 |
| Full mesh links | n(n−1)/2 |
| TCP timeout | EstRTT + 4 × DevRTT |
| Max TCP throughput | Window / RTT |
| Sequence wrap time | 2³² bytes / bandwidth (bytes/s) |

### Ports
FTP 20/21 · SSH 22 · Telnet 23 · SMTP 25 (587) · DNS 53 · DHCP 67/68 · TFTP 69 · HTTP 80 · POP3 110 · NTP 123 · IMAP 143 · SNMP 161 · BGP 179 · HTTPS 443 · RIP 520 · MySQL 3306 · RDP 3389

**UDP users:** DNS, DHCP, TFTP, SNMP, NTP, RIP, QUIC/HTTP/3, streaming, VoIP, gaming.

### Addresses
- Classes by first octet: A 1–126 · B 128–191 · C 192–223 · D 224–239 (multicast) · E 240–255.
- Private: 10/8 · 172.16/12 · 192.168/16. Loopback 127/8 (IPv6 `::1`). APIPA 169.254/16.
- MAC 48 bits · IPv4 32 bits · IPv6 128 bits · Port 16 bits.

### Headers
IPv4 20–60 B (IHL in 4-byte words, offset in 8-byte units, TTL, protocol 1/6/17) · IPv6 40 B fixed, no checksum · TCP 20–60 B · UDP 8 B · Ethernet payload 46–1500 B, frame 64–1518 B · MSS usually 1460 B.

### TCP essentials
- Handshake: SYN → SYN+ACK → ACK. Close: FIN → ACK → FIN → ACK. Active closer waits in TIME_WAIT (2 MSL).
- SYN and FIN consume a sequence number. ACK = next byte expected.
- Fast retransmit on 3 dup ACKs. Karn: no RTT samples from retransmits.
- Flow control = rwnd (receiver). Congestion control = cwnd (sender). Send ≤ min(cwnd, rwnd).
- Slow start (exponential) → ssthresh → congestion avoidance (+1 MSS/RTT).
- Timeout: ssthresh = cwnd/2, cwnd = 1 (Tahoe and Reno). 3 dup ACKs: Tahoe cwnd = 1; Reno cwnd = cwnd/2 (fast recovery).

### Domains
Hub: 1 collision, 1 broadcast · Switch: per-port collision, 1 broadcast · Router: per-interface both.

### Routing
DV = Bellman-Ford, neighbours, count-to-infinity (RIP, hop count ≤ 15, UDP 520). LS = Dijkstra, flood to all (OSPF, cost by bandwidth, IP proto 89). BGP = path vector between ASes, policy-based, TCP 179. Longest prefix match wins.

### Application
DNS: root → TLD → authoritative; A, AAAA, CNAME, MX, NS, PTR, TXT; UDP 53 (TCP for zone transfer).
HTTP: safe = GET/HEAD/OPTIONS; idempotent = + PUT/DELETE; POST and PATCH not idempotent. 401 unauthenticated vs 403 forbidden. 502 bad upstream vs 504 upstream timeout.
HTTP/1.1 keep-alive → HTTP/2 multiplexing on TCP → HTTP/3 QUIC on UDP.
TLS: asymmetric for authentication and key agreement, symmetric for data. TLS 1.3 = 1 RTT.
Email: SMTP sends, POP3 downloads, IMAP syncs. FTP: control 21, data 20 (active) — passive is firewall-friendly.

### Security one-liners
CIA triad · symmetric fast / asymmetric key exchange · hash ≠ encryption ≠ encoding · XSS runs attacker script, CSRF forges user requests · SQLi → prepared statements · SYN flood → SYN cookies · ARP spoofing → MITM on LAN · IPSec = network layer, TLS = transport.

### The flagship answer (compressed)
URL parse → DNS → ARP for gateway → TCP handshake → TLS handshake → HTTP request → NAT + routing hop by hop → load balancer/CDN → server builds response → HTTP response → browser renders and fetches sub-resources → keep-alive / close.
