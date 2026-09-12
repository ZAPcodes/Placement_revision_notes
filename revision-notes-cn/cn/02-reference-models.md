# 02 — Reference Models: OSI & TCP/IP ★★★

## Why layers
Each layer does one job and offers services to the layer above, using the layer below. You can change one layer (e.g. Wi-Fi instead of Ethernet) without changing the others. Protocols at the **same layer** on two machines talk logically to each other (**peer-to-peer communication**).

## OSI model (7 layers)
Mnemonic (top → bottom): **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing. (Bottom → top: **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way.)

| # | Layer | Job | PDU | Addressing | Protocols / examples | Devices |
|---|---|---|---|---|---|---|
| 7 | **Application** | Network services for user applications | Data | — | HTTP, FTP, SMTP, DNS, SSH, DHCP | Gateway (application-level) |
| 6 | **Presentation** | Translation, encryption, compression (data format) | Data | — | TLS/SSL*, JPEG, ASCII, MPEG | — |
| 5 | **Session** | Establish, manage, terminate sessions; checkpoints (sync) | Data | — | NetBIOS, RPC, PPTP | — |
| 4 | **Transport** | Process-to-process delivery, segmentation, reliability, flow & congestion control | **Segment** (TCP) / **Datagram** (UDP) | **Port** numbers | TCP, UDP | Firewall (L4), load balancer (L4) |
| 3 | **Network** | Host-to-host delivery across networks, logical addressing, routing | **Packet** | **IP** address | IP, ICMP, IGMP, IPSec, OSPF, RIP | **Router**, L3 switch |
| 2 | **Data Link** | Node-to-node delivery on one link, framing, MAC, error detection, access control | **Frame** | **MAC** address | Ethernet, PPP, 802.11 MAC, ARP** | **Switch**, bridge, NIC |
| 1 | **Physical** | Transmitting raw bits over the medium | **Bits** | — | Cables, signals, RS-232 | **Hub**, repeater, modem, cables |

\* TLS is usually placed at Presentation in OSI, but in practice it runs between Application and Transport (TCP/IP has no presentation layer).
\** ARP is a boundary protocol between L2 and L3 — most MCQs answer **Data Link** (sometimes "Network"). Read the options.

> [!WARNING]
> **Tricky:**
> - **Delivery scopes:** Data Link = **node-to-node (hop-to-hop)**, Network = **host-to-host (end-to-end machines)**, Transport = **process-to-process** (via ports).
> - **Error control** happens at both L2 (per link, e.g. CRC in Ethernet) and L4 (end-to-end, TCP). Why both? A frame can be error-free on each link but corrupted *inside* a router, or lost when a router drops it — only end-to-end checks catch that.
> - **ICMP** is a network-layer protocol even though it's carried inside IP packets.
> - **Routing protocols:** OSPF runs directly over IP (network layer); RIP uses UDP; BGP uses TCP — but all of them serve the network layer's routing function.
> - **DHCP and DNS** are application-layer protocols (they use UDP).
> - A **hub** is physical-layer (it just repeats bits); a **switch** is data-link; a **router** is network-layer; a **gateway** can work at all layers (protocol conversion).

## TCP/IP model
The model the Internet actually uses. Taught as 4 layers (original) or 5 layers (Kurose/Tanenbaum hybrid).

| TCP/IP (4-layer) | TCP/IP (5-layer) | OSI equivalent |
|---|---|---|
| Application | Application | Application + Presentation + Session |
| Transport (Host-to-Host) | Transport | Transport |
| Internet | Network | Network |
| Network Access (Link) | Data Link + Physical | Data Link + Physical |

## OSI vs TCP/IP

| | OSI | TCP/IP |
|---|---|---|
| Layers | 7 | 4 (or 5) |
| Nature | Theoretical reference model, designed before protocols | Practical, built around existing protocols |
| Session/Presentation | Separate layers | Folded into application |
| Transport | Connection-oriented and connectionless | Both (TCP and UDP) |
| Network layer | Both connection-oriented and connectionless | Connectionless only (IP) |
| Usage | Teaching and troubleshooting vocabulary ("it's a Layer 2 issue") | The real Internet |

## Encapsulation & decapsulation
Going **down** the stack at the sender, each layer adds its own header (Data Link also adds a trailer):

```
Application:  [ Data ]
Transport:    [ TCP hdr | Data ]                           ← segment
Network:      [ IP hdr | TCP hdr | Data ]                  ← packet
Data Link:    [ Eth hdr | IP hdr | TCP hdr | Data | FCS ]  ← frame
Physical:     1011010010...                                ← bits
```
The receiver strips headers going **up** (decapsulation). Routers decapsulate only up to Layer 3 and re-encapsulate with a new Layer-2 header for the next link.

> [!WARNING]
> **Tricky — what changes hop by hop?** At each router, the **MAC addresses change** (new source/destination for each link), but the **IP addresses stay the same** end to end (unless NAT rewrites them). TTL decreases and the IP checksum is recomputed.
