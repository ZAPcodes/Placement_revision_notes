# 05 — Network Layer: Addressing ★★★

**Job of the network layer:** deliver packets **host-to-host** across multiple networks — logical addressing (IP), routing (choosing the path), forwarding (moving a packet to the right output), and fragmentation.

## IPv4 address
- **32 bits**, written as four decimal octets: `192.168.10.25`. About 4.3 billion addresses (2³²).
- Split into a **network part** and a **host part**. The **subnet mask** marks which bits are network bits.

## Classful addressing (historical, still asked)

| Class | First octet | Leading bits | Default mask | Networks | Hosts per network | Use |
|---|---|---|---|---|---|---|
| A | 1–126 | `0` | 255.0.0.0 (/8) | 126 | 2²⁴ − 2 | Very large networks |
| B | 128–191 | `10` | 255.255.0.0 (/16) | 2¹⁴ | 2¹⁶ − 2 | Medium |
| C | 192–223 | `110` | 255.255.255.0 (/24) | 2²¹ | 254 | Small |
| D | 224–239 | `1110` | — | — | — | **Multicast** |
| E | 240–255 | `1111` | — | — | — | Reserved / experimental |

> [!WARNING]
> **Tricky:**
> - **127.x.x.x is loopback** (127.0.0.1 = localhost), so class A usable networks are 1–126. `0.0.0.0` means "this host / any address" (default route `0.0.0.0/0`).
> - **Why "− 2" hosts?** The all-0s host part is the **network address** and the all-1s host part is the **broadcast address**; neither can be assigned to a host.
> - Classful addressing wasted addresses (a company needing 300 hosts got a class B with 65,534). **CIDR** replaced it.

## Special & private addresses

| Range | Purpose |
|---|---|
| `10.0.0.0/8` | Private |
| `172.16.0.0/12` (172.16–172.31) | Private |
| `192.168.0.0/16` | Private |
| `127.0.0.0/8` | Loopback |
| `169.254.0.0/16` | **APIPA / link-local** — self-assigned when DHCP fails |
| `255.255.255.255` | Limited broadcast (this network only; routers never forward) |
| `224.0.0.0/4` | Multicast |

**Private addresses** are not routable on the public Internet; they reach it through **NAT**.

## CIDR (Classless Inter-Domain Routing)
- Address written as `a.b.c.d/n`, where **n = number of network (prefix) bits**. Any n from 0 to 32, not just 8/16/24.
- **Hosts per block** = `2^(32−n) − 2`. **Block size** = `2^(32−n)`.
- **Supernetting / route aggregation:** combine contiguous blocks into one larger prefix to shrink routing tables (e.g. four /24s → one /22).

### Subnet mask quick table

| Prefix | Mask (last octet) | Block size | Usable hosts |
|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | .128 | 128 | 126 |
| /26 | .192 | 64 | 62 |
| /27 | .224 | 32 | 30 |
| /28 | .240 | 16 | 14 |
| /29 | .248 | 8 | 6 |
| /30 | .252 | 4 | 2 (point-to-point links) |
| /31 | .254 | 2 | 2 (special: P2P links, RFC 3021) |
| /32 | .255 | 1 | Single host route |

## Subnetting method (do this in the exam)
1. Find the **interesting octet** — where the mask is neither 255 nor 0.
2. **Block size** = 256 − mask value in that octet.
3. **Network address** = largest multiple of the block size ≤ the IP's value in that octet.
4. **Broadcast** = next network address − 1.
5. **Usable range** = network + 1 to broadcast − 1.

**Worked example 1:** `172.16.45.200/20`
- /20 → mask `255.255.240.0`; interesting octet = 3rd; block size = 256 − 240 = **16**.
- Multiples of 16: ..., 32, **48** → 45 lies in 32–47.
- Network **172.16.32.0**, broadcast **172.16.47.255**, usable hosts **172.16.32.1 – 172.16.47.254**, count = 2¹² − 2 = **4094**.

**Worked example 2:** split `192.168.1.0/24` into 4 equal subnets.
- Need 2 extra bits (2² = 4) → **/26**, block size 64.
- Subnets: `.0/26`, `.64/26`, `.128/26`, `.192/26`, each with **62** usable hosts.

> [!WARNING]
> **Tricky:**
> - "How many subnets with n borrowed bits?" → **2ⁿ** (modern). Old textbooks say 2ⁿ − 2 (excluding subnet-zero and all-ones subnet) — use whatever the options imply, default to 2ⁿ.
> - "Are two hosts on the same subnet?" → AND each IP with the mask; same result = same subnet.
> - Choosing a prefix for **h hosts**: smallest n with `2^(32−n) − 2 ≥ h`. 100 hosts → /25 (126), not /26 (62).

### VLSM (Variable Length Subnet Masking) (+)
Give each subnet a mask sized to its need instead of equal sizes. **Allocate the largest subnets first** to avoid overlaps.
Example from `192.168.1.0/24` for needs of 100, 50, 20, 2 hosts: `.0/25` (126), `.128/26` (62), `.192/27` (30), `.224/30` (2).

## IPv4 header (20–60 bytes)

| Field | Bits | Notes |
|---|---|---|
| Version | 4 | 4 |
| IHL (header length) | 4 | In **4-byte words** → value 5 = 20 bytes, max 15 = 60 bytes |
| DSCP/ToS + ECN | 8 | Priority / QoS |
| Total length | 16 | Header + data, in bytes → max **65,535** |
| Identification | 16 | Same for all fragments of one datagram |
| Flags | 3 | Reserved, **DF** (Don't Fragment), **MF** (More Fragments) |
| Fragment offset | 13 | In **units of 8 bytes** |
| TTL | 8 | Decremented at each router; packet dropped at 0 (prevents loops) |
| Protocol | 8 | 1 = ICMP, 6 = TCP, 17 = UDP |
| Header checksum | 16 | Header only; recomputed at every hop (TTL changes) |
| Source / Destination IP | 32 + 32 | |
| Options | 0–40 B | Rarely used |

## Fragmentation
When a packet exceeds the **MTU** of the next link, the router splits it (IPv4 only; IPv6 routers never fragment).
- Each fragment carries the **same Identification**; **offset** = position of its data ÷ 8; **MF = 1** on all but the last fragment.
- Fragment data size (except last) must be a **multiple of 8**.
- **Reassembly happens only at the final destination**, never at intermediate routers.
- If **DF = 1** and the packet is too big, the router drops it and sends ICMP "Fragmentation Needed" (the basis of **Path MTU Discovery**).

**Worked example:** 4000-byte datagram (20 header + 3980 data), MTU 1500.
- Max data per fragment = 1500 − 20 = 1480 (divisible by 8 ✓).
- Fragments: data **1480, 1480, 1020** → total lengths 1500, 1500, 1040.
- Offsets: **0, 185, 370** (1480/8 = 185). MF: **1, 1, 0**.

> [!WARNING]
> **Tricky:** offset is in **8-byte units**, not bytes. The number of fragments counts **data** bytes; each fragment gets its own 20-byte header. If one fragment is lost, the **entire datagram** is discarded (IP doesn't retransmit).

## IPv6
- **128-bit** addresses, written in 8 groups of hex: `2001:0db8:0000:0000:0000:ff00:0042:8329` → shortened `2001:db8::ff00:42:8329` (drop leading zeros; `::` replaces one run of zero groups, **only once**).
- **Fixed 40-byte base header** with optional **extension headers** → faster router processing.
- **No header checksum** (link and transport layers already check).
- **No fragmentation by routers** — only the source fragments (using Path MTU Discovery); minimum MTU 1280 bytes.
- **No broadcast** — uses multicast and **anycast** instead.
- **Auto-configuration (SLAAC)** — hosts can configure their own address without DHCP.
- IPSec support built into the design; **Hop Limit** replaces TTL; **Flow Label** for QoS.
- Loopback `::1`; link-local `fe80::/10`.

| | IPv4 | IPv6 |
|---|---|---|
| Size | 32 bits | 128 bits |
| Notation | Dotted decimal | Hex, colon-separated |
| Header | 20–60 B, variable | 40 B fixed + extensions |
| Checksum | Yes | No |
| Fragmentation | Sender and routers | Sender only |
| Broadcast | Yes | No (multicast/anycast) |
| Configuration | Manual or DHCP | SLAAC or DHCPv6 |
| ARP | ARP | **NDP** (Neighbor Discovery, via ICMPv6) |

**Transition mechanisms:** dual stack (run both), tunnelling (IPv6 inside IPv4), translation (NAT64).

## NAT (Network Address Translation)
Lets many devices with **private** IPs share one or a few **public** IPs.
- The router rewrites the **source IP (and port)** of outgoing packets to its public IP and records the mapping in a **NAT table**; incoming replies are translated back.
- **Static NAT:** one private ↔ one public (fixed). **Dynamic NAT:** private → one of a pool of public IPs. **PAT / NAT overload:** many private → **one** public IP, distinguished by **port numbers** (what home routers do).
- **Benefits:** conserves IPv4 addresses; hides internal addressing (some security by obscurity).
- **Drawbacks:** breaks true end-to-end connectivity; inbound connections need **port forwarding**; complicates P2P, VoIP, and IPSec; adds per-connection state to the router.

> [!WARNING]
> **Tricky:** NAT modifies the IP header (and TCP/UDP ports), so it must recompute the **IP and TCP/UDP checksums**. That's a network device touching transport-layer data — a classic "layer violation" talking point.

## DHCP (Dynamic Host Configuration Protocol)
Automatically gives a host an **IP address, subnet mask, default gateway, DNS server**, and a **lease time**.
Application-layer protocol over **UDP** — server port **67**, client port **68**.

**DORA:**
1. **Discover** — client **broadcasts** (it has no IP yet; source 0.0.0.0, destination 255.255.255.255).
2. **Offer** — server offers an address.
3. **Request** — client broadcasts which offer it accepts (so other servers withdraw theirs).
4. **Acknowledge** — server confirms; the lease begins.

- The client renews at **50%** of the lease (T1) and tries other servers at 87.5% (T2).
- **DHCP relay agent:** a router that forwards DHCP broadcasts to a server on another subnet (broadcasts don't cross routers).
- If no server responds, the host self-assigns an **APIPA** address (169.254.x.x).
- **Attacks:** rogue DHCP server (hands out a malicious gateway/DNS), DHCP starvation (exhausts the pool).
