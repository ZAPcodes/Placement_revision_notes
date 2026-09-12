# 09 — Network Devices & Infrastructure ★★

## Device comparison (asked in almost every interview)

| Device | Layer | Uses | Intelligence | Collision domains | Broadcast domains |
|---|---|---|---|---|---|
| **Repeater** | 1 | Signal | Regenerates signal, extends distance | 1 | 1 |
| **Hub** | 1 | Signal | Repeats to **all** ports | 1 | 1 |
| **Bridge** | 2 | MAC | Filters between two LAN segments | Per port | 1 |
| **Switch** | 2 | MAC | Multi-port bridge, forwards to the **specific** port | Per port | 1 (per VLAN) |
| **Router** | 3 | IP | Connects **different networks**, chooses paths | Per interface | Per interface |
| **Layer-3 switch** | 2 + 3 | MAC + IP | Hardware routing between VLANs | Per port | Per VLAN |
| **Gateway** | Up to 7 | Any | Connects networks with **different protocols**, translating between them | — | — |
| **Modem** | 1 | Signal | Converts digital ↔ analog (MOdulator-DEModulator) for phone/cable/fibre lines | — | — |
| **Access point** | 2 | MAC | Bridges wireless clients to the wired LAN | — | — |
| **NIC** | 1–2 | MAC | Connects a host to the network; holds the MAC address | — | — |

> [!WARNING]
> **Tricky:**
> - **Hub vs switch:** a hub broadcasts every frame to every port (half-duplex, one collision domain); a switch learns MACs and forwards selectively (full-duplex, collision domain per port).
> - **Switch vs router:** a switch connects devices **within** a network using MAC addresses; a router connects **different** networks using IP addresses and blocks broadcasts.
> - **"Default gateway"** in host settings means the **router** interface on your subnet — the term is overloaded.
> - A home "Wi-Fi router" is really several devices in one box: router + switch + access point + NAT + DHCP server + firewall (+ modem).

## Proxy vs reverse proxy (+)

| | Forward proxy | Reverse proxy |
|---|---|---|
| Sits in front of | **Clients** | **Servers** |
| Acts on behalf of | Clients — server sees the proxy's IP | Servers — client thinks the proxy *is* the server |
| Uses | Content filtering, anonymity, caching for an org, bypassing geo-blocks | **Load balancing**, TLS termination, caching, compression, DDoS protection, hiding backend servers |
| Examples | Corporate / school proxy, Squid | **Nginx**, HAProxy, Cloudflare, AWS ALB |

## Load balancer (+)
Distributes incoming traffic across multiple servers for **scalability** and **availability**.
- **L4 load balancer:** decides using IP + port (transport layer); fast, doesn't read content.
- **L7 load balancer:** reads the HTTP request (URL, headers, cookies) → content-based routing (`/api` to one pool, `/images` to another), TLS termination, sticky sessions.
- **Algorithms:** round robin, weighted round robin, least connections, least response time, IP hash (same client → same server).
- **Health checks** remove failed servers from rotation.

## Firewall
Controls traffic between networks based on rules (usually between a trusted inside and untrusted outside).

| Type | Inspects | Notes |
|---|---|---|
| **Packet filtering (stateless)** | Each packet's IPs, ports, protocol (L3/L4) | Fast, simple; can't tell if a packet belongs to a valid connection |
| **Stateful inspection** | Tracks connection state (state table) | Allows return traffic of connections started inside; the common default |
| **Application-level / proxy firewall** | Application data (L7) | Deep inspection, slower |
| **Next-generation firewall (NGFW)** | App awareness, intrusion prevention, TLS inspection | Modern enterprise |
| **WAF (Web Application Firewall)** | HTTP traffic | Blocks SQL injection, XSS against web apps |

- **DMZ (demilitarised zone):** a separate network segment for public-facing servers (web, mail) between the outer and inner firewalls — if a public server is compromised, the internal network is still protected.
- **IDS vs IPS:** an Intrusion **Detection** System monitors and alerts; an Intrusion **Prevention** System sits inline and blocks.

## VPN (Virtual Private Network) (+)
Creates an **encrypted tunnel** over a public network, making a remote device or site behave as if on the private network.
- **Tunnelling:** encapsulating one packet inside another (e.g. a private IP packet inside an encrypted public IP packet).
- **Site-to-site VPN:** connects two office networks (typically IPSec). **Remote-access VPN:** connects individual users (SSL/TLS VPNs, WireGuard, OpenVPN).
- **Protocols:** IPSec (network layer), SSL/TLS VPN, WireGuard. Old: PPTP (insecure).

## CDN (Content Delivery Network) (+)
A geographically distributed network of **edge servers** that cache content close to users.
- Users are routed to the nearest edge via **DNS (geo-DNS)** or **anycast**.
- Cuts latency, reduces load on the origin server, absorbs traffic spikes and DDoS.
- Best for static content (images, CSS, JS, video); dynamic content can also be accelerated.
- Examples: Cloudflare, Akamai, AWS CloudFront.
