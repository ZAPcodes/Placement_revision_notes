# 06 — Network Layer: Routing ★★

## Routing vs forwarding
- **Forwarding (data plane):** moving an arriving packet to the correct output port by looking up the forwarding table. Happens per packet, in nanoseconds, in hardware.
- **Routing (control plane):** computing the paths and filling the table. Happens in the background using routing algorithms and protocols.

## Routing table & longest prefix match
Each entry: destination prefix → next hop / exit interface (+ metric).
When several prefixes match a destination, the router picks the **most specific (longest) prefix**.

> [!WARNING]
> **Tricky:** destination `192.168.1.77` with entries `192.168.0.0/16 → A`, `192.168.1.0/24 → B`, `192.168.1.64/26 → C`, `0.0.0.0/0 → D` → goes to **C** (/26 is longest). The **default route** `0.0.0.0/0` matches everything and is used only when nothing else does.

## Static vs dynamic routing

| | Static | Dynamic |
|---|---|---|
| Configuration | Manually by admin | Routers learn and exchange routes via protocols |
| Adapts to failures | No | Yes |
| Overhead | None | Bandwidth and CPU for updates |
| Use | Small or stub networks, default routes | Medium to large networks |

## Distance Vector routing
Each router knows only its **distance to every destination** and shares this vector **with its neighbours** periodically. Routes are computed with the **Bellman-Ford** equation:
`D(x, y) = min over neighbours v of { cost(x, v) + D(v, y) }`
"Routing by rumour" — a router trusts what neighbours tell it.

**Count-to-infinity problem:** when a link fails, two routers can keep telling each other they have a path through the other, and the distance climbs slowly by 1 each exchange. Good news travels fast; **bad news travels slowly**.
**Fixes:**
- **Split horizon:** don't advertise a route back to the neighbour you learned it from.
- **Poison reverse:** advertise it back with distance = infinity.
- **Hold-down timers** and a **maximum hop count** (RIP: 16 = infinity).

**RIP (Routing Information Protocol):** distance vector, metric = **hop count**, max **15** hops (16 = unreachable), updates every **30 s**, runs over **UDP port 520**. Simple but slow to converge; only for small networks.

## Link State routing
Each router learns the **full topology**:
1. Discover neighbours (hello packets) and measure link costs.
2. **Flood** a Link State Advertisement (LSA) about its own links to **all** routers.
3. Each router builds the same map and runs **Dijkstra's algorithm** to find shortest paths to everyone.

**OSPF (Open Shortest Path First):** link state; cost based on **bandwidth**; fast convergence; supports **areas** (Area 0 is the backbone) to limit flooding; sends updates only on change (plus periodic refresh); runs **directly over IP (protocol 89)**, not TCP/UDP. The standard interior protocol in enterprises.

## Distance Vector vs Link State

| | Distance Vector (RIP) | Link State (OSPF) |
|---|---|---|
| Knowledge | Only neighbours' distance vectors | Full topology map |
| Algorithm | Bellman-Ford | Dijkstra |
| Shares | Whole table, with **neighbours** only | Own link info, with **all** routers |
| Updates | Periodic | Triggered by changes |
| Convergence | Slow; count-to-infinity | Fast |
| Resources | Low CPU/memory | Higher CPU/memory |
| Scale | Small networks | Large networks |

## Interior vs exterior routing
- **Autonomous System (AS):** a network under one administration (an ISP, a large company), identified by an **ASN**.
- **IGP (Interior Gateway Protocol):** routing **within** an AS — RIP, OSPF, IS-IS, EIGRP (Cisco hybrid).
- **EGP (Exterior Gateway Protocol):** routing **between** ASes — **BGP** is the only one used today.

### BGP (Border Gateway Protocol)
- **Path vector** protocol: advertises the full **AS path** to each prefix → loop prevention (reject any path containing your own AS).
- Chooses routes based on **policies** (business relationships, cost), not just shortest distance.
- Runs over **TCP port 179** (needs reliable delivery of large route tables).
- **eBGP** between ASes, **iBGP** within one AS. "The glue of the Internet."
- BGP hijacks/leaks (a network wrongly announcing others' prefixes) have caused major outages.

## Other routing concepts
- **Administrative distance:** a router's trust ranking between protocols when two offer routes to the same prefix (connected < static < OSPF < RIP).
- **Hierarchical routing:** divide networks into regions/areas to keep routing tables small.
- **Flooding:** send every packet out every link except the arriving one — robust but wasteful; controlled with sequence numbers or hop counts.

## ICMP (Internet Control Message Protocol)
Network-layer helper protocol (carried inside IP, protocol number 1) for **error reporting and diagnostics** — it doesn't carry user data.

| Message | Use |
|---|---|
| Echo Request / Echo Reply (types 8 / 0) | **ping** |
| Destination Unreachable (type 3) | Network/host/port unreachable, fragmentation needed |
| Time Exceeded (type 11) | TTL reached 0 → used by **traceroute** |
| Redirect (type 5) | Tells host there's a better next hop |

**How traceroute works:** send packets with **TTL = 1, 2, 3, ...**. Each router that drops a packet because TTL hit 0 sends back **ICMP Time Exceeded**, revealing its address. The destination finally replies (ICMP Echo Reply, or Port Unreachable for UDP-based traceroute on Linux). Windows `tracert` uses ICMP echo; Linux `traceroute` uses UDP by default.

> [!WARNING]
> **Tricky:**
> - ICMP errors are **never** sent about ICMP error messages (prevents error storms), nor for fragments other than the first, nor for broadcast/multicast packets.
> - `ping` working proves Layer 3 reachability only — it says nothing about whether port 80 is open. Many firewalls block ICMP, so a failed ping doesn't prove the host is down.

## Router vs Layer-3 switch (+)
Both route by IP. A **Layer-3 switch** does routing in hardware (ASICs) at wire speed between VLANs inside a LAN, with many Ethernet ports. A **router** supports more WAN interfaces and features (NAT, VPN, complex routing protocols, firewalling) and connects different networks/ISPs.
