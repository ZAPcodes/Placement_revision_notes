# 01 — Fundamentals ★★

## What is a network
A set of devices (**nodes**: computers, phones, routers, printers) connected by **links** (wired or wireless) to share data and resources. The **Internet** is a network of networks.

## Network types by scale

| Type | Span | Example |
|---|---|---|
| PAN | A few metres | Bluetooth headset ↔ phone |
| LAN | Building / campus | Office or hostel network |
| MAN | City | Cable TV network, city-wide Wi-Fi |
| WAN | Country / world | The Internet, bank networks |

- **Internet:** public, global. **Intranet:** private network inside one organisation using internet technologies. **Extranet:** intranet extended to selected outside parties (partners, vendors).

## Topologies

| Topology | Structure | Pros | Cons |
|---|---|---|---|
| **Bus** | All nodes on one backbone cable | Cheap, little cable | Backbone failure kills everything; collisions; hard to troubleshoot |
| **Star** | All nodes to a central hub/switch | Easy to add/troubleshoot; one cable failing affects one node | Central device is a single point of failure |
| **Ring** | Each node connects to two neighbours in a loop | Orderly access (token passing), no collisions | One break can take the ring down (unless dual ring) |
| **Mesh** | Every node connects to every other (full) or many (partial) | Very reliable, redundant paths, private links | Expensive, lots of cabling and ports |
| **Tree** | Hierarchy of stars | Scalable, easy to extend | Root failure affects the branch below |
| **Hybrid** | Combination | Flexible | Complex |

> [!WARNING]
> **Tricky — full mesh counts:** for `n` nodes, links = **n(n−1)/2** (duplex) and each node needs **n−1** ports. 10 nodes → 45 links.
> Star needs **n** links (to the hub); ring needs **n**; bus needs 1 backbone + n drop lines.

## Transmission modes
- **Simplex:** one direction only (keyboard → computer, TV broadcast).
- **Half-duplex:** both directions, one at a time (walkie-talkie, hub-based Ethernet).
- **Full-duplex:** both directions simultaneously (phone call, switched Ethernet).

## Switching techniques

| | Circuit switching | Packet switching | Message switching |
|---|---|---|---|
| Path | Dedicated path reserved for the whole call | No reservation; each packet shares links | Whole message stored and forwarded |
| Setup | Needed before data flows | Not needed (datagram) | Not needed |
| Resource use | Wasteful when idle | Efficient (statistical multiplexing) | Needs large storage at nodes |
| Delay | Constant after setup | Variable (queuing) | High |
| Example | Traditional telephone network | Internet | Old telegraph |

**Packet switching has two flavours:**
- **Datagram:** each packet routed independently; may arrive out of order (IP).
- **Virtual circuit:** a logical path set up first, all packets follow it in order, but links aren't exclusively reserved (ATM, MPLS, Frame Relay).

## Performance: delays and speed

| Delay | Formula | Depends on |
|---|---|---|
| **Transmission (Tt)** | `L / B` (packet length ÷ bandwidth) | Packet size, link speed |
| **Propagation (Tp)** | `d / v` (distance ÷ signal speed) | Distance, medium (≈ 2×10⁸ m/s in cable/fibre) |
| **Queuing** | Variable | Congestion at routers |
| **Processing** | Usually tiny | Router header processing |

**Total delay per hop** = Tt + Tp + Tqueue + Tproc.

> [!WARNING]
> **Tricky:**
> - **Transmission delay ≠ propagation delay.** Tt is the time to push all bits *onto* the wire; Tp is the time for one bit to *travel* the wire. A faster link reduces Tt, never Tp.
> - Units: bandwidth in **bits**/sec, file sizes often in **bytes** — multiply by 8. "1 KB" in networking numericals may mean 1000 or 1024 bytes — check the question.
> - With `n` store-and-forward hops (n links), end-to-end delay for one packet = `n × (Tt + Tp)` (ignoring queuing).

**Worked example:** 1000-byte packet, 1 Mbps link, 2000 km, signal speed 2×10⁸ m/s.
Tt = 8000 bits / 10⁶ = **8 ms**. Tp = 2×10⁶ m / 2×10⁸ = **10 ms**. Total (one hop) = **18 ms**.

- **Bandwidth:** maximum data rate of a link. **Throughput:** actual achieved rate (≤ bandwidth). **Latency:** time for data to go from source to destination.
- **RTT (round-trip time):** time for a signal to go and a response to come back ≈ 2 × Tp (plus processing).
- **Bandwidth–delay product** = bandwidth × Tp = the number of bits "in flight" filling the pipe. It decides how big a sliding window must be to keep the link busy.

## Communication types
- **Unicast:** one-to-one. **Broadcast:** one-to-all in the network. **Multicast:** one-to-a-group (IPTV, class D addresses). **Anycast:** one-to-nearest of a group (DNS root servers, CDNs; used in IPv6).

## Architectures
- **Client–server:** central server provides services; clients request (web, email). Easy to manage; server can be a bottleneck.
- **Peer-to-peer (P2P):** every node is both client and server (BitTorrent). Scales naturally; harder to secure and manage.
