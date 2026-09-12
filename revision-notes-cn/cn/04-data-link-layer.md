# 04 — Data Link Layer ★★★

**Job:** reliable **node-to-node** delivery over a single link. Responsibilities: framing, physical (MAC) addressing, error detection/correction, flow control, and controlling access to a shared medium.

**Sublayers (IEEE 802):** **LLC** (Logical Link Control — flow and error control, interfaces with network layer) and **MAC** (Media Access Control — addressing and medium access).

## Framing
Split the bit stream into **frames** so the receiver knows where each begins and ends.
- **Character count:** header says frame length (one corrupted count ruins sync).
- **Byte stuffing:** flag byte marks boundaries; an escape byte (ESC) is inserted before any flag/ESC inside data.
- **Bit stuffing:** flag `01111110`; after five consecutive 1s in the data, the sender inserts a 0 (HDLC).

> [!WARNING]
> **Tricky — bit stuffing:** data `011111110` → stuffed `0111110110` (a 0 inserted after the first five 1s). The receiver removes any 0 that follows five 1s.

## MAC address
- **48 bits** (6 bytes), written as `AA:BB:CC:DD:EE:FF`. First 24 bits = **OUI** (manufacturer), last 24 = device-specific.
- Burned into the NIC (though software can change it). **Broadcast MAC:** `FF:FF:FF:FF:FF:FF`.
- **MAC vs IP:** MAC is a physical/hardware address, flat, used within one link; IP is a logical, hierarchical address used across networks. MAC changes hop by hop in a packet's journey; IP doesn't.

## Error detection

| Method | How | Detects |
|---|---|---|
| **Single parity** | Add 1 bit so the count of 1s is even (or odd) | All **odd** numbers of bit errors; misses even numbers |
| **2-D parity** | Parity per row and per column | All 1-, 2-, 3-bit errors; **corrects** single-bit errors |
| **Checksum** | Sum data in 16-bit words (1's complement), send complement of the sum | Used in IP, TCP, UDP; weaker than CRC |
| **CRC** | Polynomial division; append remainder | All single-bit, double-bit, odd-count errors, and all burst errors ≤ degree of generator. Used in Ethernet |

### CRC procedure
1. Generator of degree `r` (r+1 bits). Append **r zeros** to the data.
2. Divide by the generator using **XOR (modulo-2)** division.
3. The r-bit **remainder** is the CRC; replace the zeros with it and send.
4. Receiver divides the whole received frame by the generator — remainder **0** means no error detected.

**Worked example:** data `1101011011`, generator `10011` (degree 4). Append `0000` → divide → remainder **`1110`**. Transmitted frame: `11010110111110`.

> [!WARNING]
> **Tricky:** append **degree** zeros (generator bits − 1), not generator-length zeros. A generator with a `+1` term (last bit 1) catches all single-bit errors; one with the factor `(x+1)` catches all odd-count errors.

## Error correction — Hamming code
- For `m` data bits, the number of redundant bits `r` must satisfy **`2^r ≥ m + r + 1`**. (m = 4 → r = 3; m = 7 → r = 4.)
- Parity bits sit at positions that are **powers of 2** (1, 2, 4, 8...). Parity bit at position p covers every position whose binary index has that bit set.
- On receipt, recompute the parity checks; the resulting binary number (**syndrome**) is the **position of the erroneous bit** (0 means no error).
- Hamming distance of the code = 3 → detects 2-bit errors, corrects 1-bit errors.

**Worked example (even parity):** data `1011` → positions 3, 5, 6, 7 hold 1, 0, 1, 1.
p1 (covers 1,3,5,7) = 0 · p2 (2,3,6,7) = 1 · p4 (4,5,6,7) = 0 → codeword **`0110011`**.
If bit 6 flips (received `0110001`), the checks give c4 c2 c1 = `110` = **6** → flip bit 6 back.

> [!WARNING]
> **Tricky — Hamming distance rules:** to **detect** `d` errors you need minimum distance `d + 1`; to **correct** `d` errors you need `2d + 1`.

## Flow control & ARQ (Automatic Repeat reQuest)
Let `a = Tp / Tt`.

| Protocol | Sender window | Receiver window | On error | Efficiency |
|---|---|---|---|---|
| **Stop-and-Wait** | 1 | 1 | Retransmit that frame after timeout | `1 / (1 + 2a)` |
| **Go-Back-N** | N ≤ `2^k − 1` | 1 | Retransmit the lost frame **and every frame after it** | `N / (1 + 2a)` (max 1) |
| **Selective Repeat** | N ≤ `2^(k−1)` | Same as sender | Retransmit **only** the lost frame; receiver buffers out-of-order frames | `N / (1 + 2a)` (max 1) |

`k` = number of bits in the sequence number field.
- GBN uses **cumulative ACKs**; SR uses **individual ACKs** (and often NAKs).
- **Optimal window** to reach 100% utilisation: `N ≥ 1 + 2a`.
- Stop-and-Wait needs only 1-bit sequence numbers (0 and 1) to detect duplicates.

> [!WARNING]
> **Tricky:**
> - Why GBN's window is at most `2^k − 1`, not `2^k`: if all 2^k frames are sent and every ACK is lost, the receiver can't tell a retransmitted old frame 0 from a new frame 0.
> - Why SR's is at most `2^(k−1)`: the receiver's window must not overlap old and new sequence numbers after a wrap-around.
> - **Minimum sequence-number bits** for a window of N: GBN → `⌈log₂(N+1)⌉`; SR → `⌈log₂(2N)⌉`.
> - Efficiency formulas assume no errors, and ACK frames of negligible size.

**Worked example:** Tt = 1 ms, Tp = 24.5 ms → a = 24.5.
Stop-and-Wait efficiency = 1 / (1 + 49) = **2%**. To reach 100%, window N ≥ 1 + 49 = **50** frames → GBN needs ⌈log₂ 51⌉ = **6** sequence bits.

## Multiple access protocols
When many nodes share one medium, who talks when?

### Random access
- **Pure ALOHA:** send whenever you like; on collision wait a random time. Vulnerable period = **2 × Tt**. Max throughput **18.4%** (`1/(2e)`) at G = 0.5.
- **Slotted ALOHA:** time divided into slots; send only at slot start. Vulnerable period = **Tt**. Max throughput **36.8%** (`1/e`) at G = 1.
- **CSMA (Carrier Sense Multiple Access):** listen before sending. Variants: **1-persistent** (send immediately when idle — most collisions), **non-persistent** (if busy, wait random time before sensing again), **p-persistent** (when idle, send with probability p; used with slotted channels).
- **CSMA/CD (Collision Detection):** listen while sending; on collision, stop, send a **jam signal**, then wait using **binary exponential back-off** (after the n-th collision pick a random `k` in `0 … 2ⁿ − 1` slots). Used in **classic wired Ethernet**.
- **CSMA/CA (Collision Avoidance):** used in **Wi-Fi**. Wait for an inter-frame gap (IFS), random back-off, optional **RTS/CTS** handshake, and an ACK for every frame.

> [!WARNING]
> **Tricky — CSMA/CD minimum frame size:** a sender must still be transmitting when news of a collision gets back, so **`Tt ≥ 2 × Tp`** → **`Lmin = 2 × Tp × B`**.
> Example: 10 Mbps, 2500 m, 2×10⁸ m/s → Tp = 12.5 µs → Lmin = 2 × 12.5 µs × 10⁷ = **250 bits**.
> Classic Ethernet's minimum frame of **64 bytes (512 bits)** comes from this rule. CSMA/CD efficiency ≈ `1 / (1 + 6.44a)`.

> [!WARNING]
> **Tricky — why Wi-Fi uses CA, not CD:** a wireless radio can't listen while it transmits (its own signal drowns everything), and because of the **hidden terminal problem** a collision at the receiver may be invisible to the sender. So it avoids collisions rather than detecting them.

### Controlled access
- **Polling:** a primary node asks each station in turn.
- **Token passing:** a special token frame circulates; only the holder may send (Token Ring, FDDI). No collisions.
- **Reservation:** stations reserve slots before sending.

### Channelisation
FDMA, TDMA, CDMA — split the channel by frequency, time, or code (see 03).

## Ethernet (IEEE 802.3)
Frame format:

| Preamble | SFD | Dest MAC | Src MAC | Type/Length | Data | FCS (CRC-32) |
|---|---|---|---|---|---|---|
| 7 B | 1 B | 6 B | 6 B | 2 B | **46–1500 B** | 4 B |

- **Frame size:** 64 to 1518 bytes (excluding preamble/SFD). Payload below 46 bytes is **padded**.
- **MTU** of Ethernet = **1500 bytes** (max payload).
- Modern switched full-duplex Ethernet has no collisions, so CSMA/CD is effectively unused.

## Switches
- Learn which MAC address lives on which port by reading the **source MAC** of incoming frames (**MAC address table / CAM table**).
- **Forward** a frame only to the destination's port if known; **flood** it to all ports (except the incoming one) if unknown; always flood broadcasts.
- **Forwarding methods:** store-and-forward (checks CRC first), cut-through (starts forwarding after reading the destination MAC — faster, may forward bad frames), fragment-free (reads first 64 bytes).

> [!WARNING]
> **Tricky — collision and broadcast domains:**
>
> | Device | Collision domains | Broadcast domains |
> |---|---|---|
> | Hub (n ports) | 1 | 1 |
> | Switch (n ports) | **n** (one per port) | **1** (unless VLANs) |
> | Router (n interfaces) | n | **n** (one per interface) |
>
> Switches split collision domains; **routers (or VLANs) split broadcast domains.**

## ARP (Address Resolution Protocol)
**Maps a known IP address to an unknown MAC address** within the same local network.
1. Host checks its ARP cache.
2. If missing, it **broadcasts** an ARP request: "Who has 192.168.1.5? Tell 192.168.1.2."
3. The owner replies with a **unicast** ARP reply containing its MAC.
4. Both cache the mapping (entries expire).

- If the destination is on **another network**, the host ARPs for its **default gateway's** MAC, not the remote host's.
- **RARP:** MAC → IP (obsolete; replaced by BOOTP and then DHCP).
- **Gratuitous ARP:** a host announces its own IP–MAC mapping (detects duplicate IPs, updates others' caches after failover).
- **ARP spoofing/poisoning:** an attacker sends fake ARP replies to link their MAC with the gateway's IP → man-in-the-middle. ARP has **no authentication**.

## VLANs (+)
- A **VLAN** splits one physical switch into multiple **logical** LANs; each VLAN is its own **broadcast domain**.
- Benefits: security (separating departments), reduced broadcast traffic, flexible grouping independent of physical location.
- **Trunk ports** carry multiple VLANs between switches, tagging frames with **IEEE 802.1Q** tags (4 bytes, 12-bit VLAN ID → up to 4094 VLANs).
- Traffic **between** VLANs needs a router or Layer-3 switch (inter-VLAN routing).

## Spanning Tree Protocol (+)
- Redundant switch links create **loops** → broadcast storms and MAC table instability (Ethernet frames have no TTL).
- **STP (IEEE 802.1D)** elects a **root bridge** (lowest bridge ID), computes the shortest paths to it, and **blocks** redundant ports so the topology becomes a loop-free tree. Blocked links activate if an active one fails. **RSTP** (802.1w) converges faster.

## PPP & HDLC (brief)
Point-to-point data link protocols for WAN links. **HDLC** is bit-oriented (uses bit stuffing, flag `01111110`). **PPP** adds authentication (PAP/CHAP) and supports multiple network-layer protocols.
