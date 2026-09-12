# 11 — Wireless & Modern Topics ★

## Wi-Fi (IEEE 802.11)
- Uses **CSMA/CA** (see 04) — every data frame is acknowledged; optional RTS/CTS.
- Bands: **2.4 GHz** (longer range, more interference, fewer channels), **5 GHz** (faster, shorter range), **6 GHz** (Wi-Fi 6E/7).
- Generations: Wi-Fi 4 (802.11n), Wi-Fi 5 (802.11ac), **Wi-Fi 6 (802.11ax)**, Wi-Fi 7 (802.11be).
- **SSID:** network name. **BSS:** one AP and its clients. **ESS:** multiple APs with the same SSID (roaming).
- **Security:** WEP (broken) → WPA → **WPA2** (AES/CCMP) → **WPA3** (SAE handshake, protects against offline password guessing).

### Hidden & exposed terminal problems
- **Hidden terminal:** A and C are both in range of B but not of each other. Both sense the channel as free and transmit to B → **collision at B** that neither sender detects. **Fix:** **RTS/CTS** — B's CTS is heard by both, so C stays quiet.
- **Exposed terminal:** B wants to send to A; C wants to send to D (away from B). C hears B and **unnecessarily defers**, though the two transmissions wouldn't interfere. Result: wasted capacity.

> [!WARNING]
> **Tricky:** RTS/CTS solves the **hidden** terminal problem; it doesn't solve the **exposed** terminal problem (it can make it slightly worse). Hidden terminal = collisions you can't detect; exposed terminal = transmissions you needlessly avoid.

## Mobile networks (brief)
1G analog voice → 2G digital voice + SMS (GSM) → 3G mobile data (UMTS) → 4G LTE all-IP → **5G**: very high bandwidth, low latency (~1 ms target), massive IoT device density, network slicing.

## SDN (Software-Defined Networking)
- **Separates the control plane from the data plane.** Switches just forward according to flow tables; a **central controller** (software) decides the rules for the whole network.
- Controller ↔ switch communication via protocols like **OpenFlow**.
- **Benefits:** centralised view and management, programmable network (automation via APIs), faster changes, easier traffic engineering.
- **NFV (Network Function Virtualisation):** run network functions (firewalls, load balancers, routers) as software on commodity servers instead of dedicated hardware.

## Cloud networking basics (+)
- **VPC (Virtual Private Cloud):** your isolated virtual network in a cloud provider, with your own IP range (CIDR).
- **Public subnet:** has a route to an **Internet Gateway** (for web servers). **Private subnet:** no direct inbound internet access (databases); outbound via a **NAT gateway**.
- **Security groups** (stateful, instance-level firewall) vs **network ACLs** (stateless, subnet-level).
- Load balancers, CDNs, and DNS services (e.g. Route 53) are managed services built on the concepts in 08–09.

## IoT protocols (name-level awareness)
**MQTT** (lightweight publish–subscribe over TCP), **CoAP** (REST-like over UDP), Zigbee, Bluetooth Low Energy.
