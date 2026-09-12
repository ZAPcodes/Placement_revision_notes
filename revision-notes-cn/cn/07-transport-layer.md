# 07 — Transport Layer: TCP & UDP ★★★

**Job:** **process-to-process** delivery. IP gets data to the right host; the transport layer gets it to the right **application** on that host, using **port numbers**.

## Ports & sockets
- **Port:** 16-bit number (0–65535) identifying an application endpoint.
  - **Well-known:** 0–1023 (need admin rights to bind). **Registered:** 1024–49151. **Dynamic/ephemeral:** 49152–65535 (clients pick these temporarily).
- **Socket:** IP address + port. A **TCP connection** is identified by the **4-tuple**: (source IP, source port, destination IP, destination port) — plus the protocol, making a 5-tuple.
- **Multiplexing** (sender): gather data from many sockets, add headers. **Demultiplexing** (receiver): deliver each segment to the right socket.

> [!WARNING]
> **Tricky:** a web server handles thousands of clients on **one port (443)** because each connection has a **different 4-tuple** (different client IP/port). UDP demultiplexes using only the destination IP + port.

### Must-know port numbers

| Port | Protocol | Transport |
|---|---|---|
| 20 / 21 | FTP data / control | TCP |
| 22 | SSH (also SCP, SFTP) | TCP |
| 23 | Telnet | TCP |
| 25 | SMTP | TCP |
| 53 | DNS | **UDP** (TCP for large responses/zone transfers) |
| 67 / 68 | DHCP server / client | UDP |
| 69 | TFTP | UDP |
| 80 | HTTP | TCP |
| 110 | POP3 | TCP |
| 123 | NTP | UDP |
| 143 | IMAP | TCP |
| 161 / 162 | SNMP / SNMP traps | UDP |
| 179 | BGP | TCP |
| 443 | HTTPS | TCP (UDP for HTTP/3) |
| 3306 | MySQL | TCP |
| 3389 | RDP | TCP |

## UDP (User Datagram Protocol)
- **Connectionless**, unreliable, no ordering, no flow or congestion control — "send and forget".
- **8-byte header:** source port, destination port, length, checksum.
- **Why use it:** no handshake latency, tiny header, no head-of-line blocking, supports broadcast/multicast, app controls timing.
- **Used by:** DNS, DHCP, streaming, VoIP, online games, SNMP, TFTP, NTP, **QUIC/HTTP/3**.

## TCP (Transmission Control Protocol)
- **Connection-oriented**, **reliable**, **in-order**, **byte-stream** (no message boundaries), **full-duplex**, with **flow control** and **congestion control**.
- **Header: 20–60 bytes.**

| Field | Purpose |
|---|---|
| Source / destination port | 16 bits each |
| **Sequence number** | Byte number of the first data byte in this segment (32 bits) |
| **Acknowledgement number** | **Next byte expected** from the other side (cumulative ACK) |
| Header length | In 4-byte words |
| Flags | **SYN, ACK, FIN, RST, PSH, URG** (+ ECE, CWR) |
| **Window** | Receiver's available buffer (flow control), 16 bits → max 65,535 without the window-scale option |
| Checksum | Over header, data, and a pseudo-header (includes IPs) |
| Urgent pointer | With URG |
| Options | MSS, window scale, SACK, timestamps |

### TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Connection | 3-way handshake | None |
| Reliability | ACKs, retransmission | None |
| Ordering | Guaranteed | Not guaranteed |
| Flow / congestion control | Yes / Yes | No / No |
| Header | 20–60 B | 8 B |
| Data unit | Byte stream | Individual datagrams (boundaries preserved) |
| Speed | Slower (overhead) | Faster |
| Broadcast/multicast | No | Yes |
| Use | Web, email, file transfer, SSH, databases | DNS, streaming, gaming, VoIP, DHCP |

> [!WARNING]
> **Tricky — "When would you choose UDP over TCP?"** When timeliness beats completeness: a late video frame or voice packet is useless, so retransmitting it is pointless. Also for short request–response exchanges (DNS: one query, one reply — a TCP handshake would triple the latency) and when the app wants to build its own reliability (QUIC does exactly that on top of UDP).

## Connection establishment — 3-way handshake
```
Client                                Server
  | --- SYN, seq = x --------------->  |   (client: SYN_SENT)
  | <-- SYN+ACK, seq = y, ack = x+1 -- |   (server: SYN_RECEIVED)
  | --- ACK, seq = x+1, ack = y+1 ---> |   (both: ESTABLISHED)
```
- Both sides pick a random **Initial Sequence Number (ISN)** — random to prevent old-segment confusion and sequence-prediction attacks.
- **SYN and FIN each consume one sequence number** even though they carry no data.

> [!WARNING]
> **Tricky:**
> - **Why 3, not 2?** Both sides must confirm the other received their ISN. With 2 steps the server can't know the client got its SYN+ACK, and a delayed duplicate SYN from an old connection could open a bogus connection on the server.
> - **What if the final ACK is lost?** The server stays in SYN_RECEIVED and retransmits SYN+ACK. Meanwhile, if the client sends data, that data segment carries the ACK and completes the handshake.
> - **SYN flood attack:** attacker sends many SYNs with spoofed IPs and never completes; the server's half-open connection queue fills up. Defence: **SYN cookies** (encode state in the ISN, allocate memory only after the final ACK).
> - Data can be carried in the third packet (ACK), but normally not in SYN (except TCP Fast Open).

## Connection termination — 4-way handshake
```
Client (active close)                  Server (passive close)
  | --- FIN -----------------------> |   client: FIN_WAIT_1
  | <-- ACK ------------------------ |   server: CLOSE_WAIT, client: FIN_WAIT_2
  |                                  |   (server may still send data — half-close)
  | <-- FIN ------------------------ |   server: LAST_ACK
  | --- ACK -----------------------> |   client: TIME_WAIT → (2 × MSL) → CLOSED
```
- Four steps because TCP is full-duplex — each direction closes independently (**half-close**). The middle ACK and FIN can be combined into one segment (making it 3-way).
- **RST** aborts a connection immediately (e.g. a segment to a closed port, or crash recovery).

> [!WARNING]
> **Tricky — TIME_WAIT:**
> - Held by the side that **closes first** (active closer) for **2 × MSL** (Maximum Segment Lifetime, typically 30 s–2 min).
> - **Why?** (1) If the final ACK is lost, the other side resends FIN; the closer must still be around to re-ACK it. (2) Lets old duplicate segments of this connection die out so they can't be mistaken for a new connection with the same 4-tuple.
> - Busy servers that actively close many connections can run out of ports stuck in TIME_WAIT.
> - **Too many CLOSE_WAIT** sockets on a server = the application isn't calling `close()` (a bug in the app, not TCP).

## Reliability mechanisms
- **Sequence numbers + cumulative ACKs** (ACK n = "I've got everything before byte n").
- **Retransmission on timeout** and **fast retransmit** after **3 duplicate ACKs** (don't wait for the timer).
- **SACK (Selective ACK) option:** receiver reports which blocks it has, so only the gaps are resent.
- **Checksum** for corruption; **reordering** in the receive buffer; **duplicates** discarded by sequence number.
- TCP is a hybrid of GBN (cumulative ACKs) and SR (buffers out-of-order data, retransmits only one segment).

### RTT estimation & timeout (+)
- `EstimatedRTT = (1 − α) × EstimatedRTT + α × SampleRTT`, α = 1/8 (exponential weighted moving average).
- `DevRTT = (1 − β) × DevRTT + β × |SampleRTT − EstimatedRTT|`, β = 1/4.
- **`Timeout = EstimatedRTT + 4 × DevRTT`**.
- **Karn's algorithm:** don't take RTT samples from **retransmitted** segments (ambiguous which transmission the ACK is for), and double the timeout after each retransmission (exponential back-off).

## Flow control
Stops a fast sender from **overwhelming the receiver's buffer**.
- The receiver advertises **rwnd** (receive window = free buffer space) in every ACK. The sender keeps unacknowledged data ≤ rwnd.
- **rwnd = 0** → sender stops and sends small **window probe** segments (persist timer) until the window opens.
- **Silly window syndrome** (+): tiny windows cause tiny segments with huge header overhead. **Receiver fix (Clark):** don't advertise a window until it's reasonably large. **Sender fix (Nagle's algorithm):** buffer small writes until the previous data is ACKed or a full MSS is ready. Nagle is often disabled (`TCP_NODELAY`) for interactive apps like games and SSH.

## Congestion control
Stops senders from **overwhelming the network** (routers' queues).
- Sender keeps a **congestion window (cwnd)**. **Effective window = min(cwnd, rwnd).**
- Congestion is inferred from **loss**: a **timeout** (severe) or **3 duplicate ACKs** (mild — later packets are still getting through).

**Phases:**
1. **Slow start:** cwnd starts at 1 MSS and **doubles every RTT** (+1 MSS per ACK) — exponential — until it reaches **ssthresh**.
2. **Congestion avoidance:** above ssthresh, cwnd grows by **1 MSS per RTT** — linear (**Additive Increase**).
3. **On loss:** **Multiplicative Decrease**:

| Event | TCP Tahoe | TCP Reno |
|---|---|---|
| **Timeout** | ssthresh = cwnd/2, **cwnd = 1**, slow start | ssthresh = cwnd/2, **cwnd = 1**, slow start |
| **3 duplicate ACKs** | ssthresh = cwnd/2, **cwnd = 1**, slow start | ssthresh = cwnd/2, **cwnd = ssthresh** (+3), **fast recovery** → congestion avoidance |

AIMD produces the classic **saw-tooth** graph of cwnd over time. Modern variants: **CUBIC** (Linux default), **BBR** (Google; models bandwidth and RTT instead of reacting to loss).

> [!WARNING]
> **Tricky — congestion window numerical:** initial ssthresh = 16, cwnd starts at 1. Timeout occurs when cwnd = 20.
> - RTT-wise cwnd: 1 → 2 → 4 → 8 → **16** (hits ssthresh) → 17 → 18 → 19 → **20** → timeout.
> - New ssthresh = 20 / 2 = **10**; cwnd = 1 → 2 → 4 → 8 → **10** (capped at ssthresh, then linear) → 11 → ...
>
> Watch for: doubling overshooting ssthresh (cap it at ssthresh), Tahoe vs Reno on 3 dup-ACKs, and whether the question counts in segments or bytes.

> [!WARNING]
> **Tricky — flow control vs congestion control:** flow control protects the **receiver** (rwnd, advertised by the receiver). Congestion control protects the **network** (cwnd, computed by the sender). Both limit the sender, using the minimum.

## Other TCP facts often asked
- **MSS (Maximum Segment Size):** largest data payload in one segment, negotiated in SYN options; typically **1460 bytes** on Ethernet (1500 MTU − 20 IP − 20 TCP).
- **Sequence number wrap-around:** a 32-bit space wraps after 4 GB; on very fast links it can wrap within one MSL → **timestamps (PAWS)** fix this. Wrap-around time = 2³² bytes ÷ bandwidth (bytes/s).
- **Keep-alive:** optional probes on idle connections to detect a dead peer.
- **Head-of-line blocking:** a lost TCP segment stalls delivery of everything after it, even data belonging to unrelated streams — the reason HTTP/3 moved to QUIC over UDP.
- **Maximum throughput of one TCP connection** ≈ window size ÷ RTT (e.g. 64 KB window, 100 ms RTT → ~5.2 Mbps, no matter how fast the link) — why window scaling exists.
