# 08 — Application Layer ★★★

## DNS (Domain Name System)
Translates human-readable names (`www.google.com`) into IP addresses. A **distributed, hierarchical database** — no single server knows everything.

### Hierarchy
```
               . (root)                    13 named root server identities (a–m), hundreds of anycast instances
        /       |        \
     .com     .org     .in                 TLD (Top-Level Domain) servers
      |
  google.com                               Authoritative name servers (owned by the domain)
      |
  www.google.com
```
Plus the **local / recursive resolver** (your ISP's, or 8.8.8.8, 1.1.1.1), which does the legwork for clients.

### Resolution
1. Browser cache → OS cache (and `hosts` file) → router cache.
2. Query the **recursive resolver**. If it has a cached answer, done.
3. Otherwise the resolver asks a **root** server → gets a referral to the `.com` TLD servers.
4. Asks the **TLD** server → gets a referral to `google.com`'s authoritative servers.
5. Asks the **authoritative** server → gets the IP.
6. Resolver caches the answer (for its **TTL**) and returns it to the client.

- **Recursive query:** "give me the final answer" — client → resolver.
- **Iterative query:** "tell me who to ask next" — resolver → root/TLD/authoritative.

### Record types

| Record | Maps |
|---|---|
| **A** | Name → IPv4 address |
| **AAAA** | Name → IPv6 address |
| **CNAME** | Alias → canonical name (`www.example.com` → `example.com`) |
| **MX** | Domain → mail server (with priority) |
| **NS** | Domain → its authoritative name servers |
| **PTR** | IP → name (**reverse DNS**) |
| **TXT** | Arbitrary text (SPF, DKIM, domain verification) |
| **SOA** | Start of authority — zone admin info, serial number |
| **SRV** | Service location (host + port) |

> [!WARNING]
> **Tricky:**
> - DNS uses **UDP port 53** for normal queries (fast, one round trip). It uses **TCP** for **zone transfers** (primary → secondary servers) and for responses too large for UDP (traditionally > 512 bytes).
> - A **CNAME** can't coexist with other records for the same name, and can't be placed at the zone apex (`example.com` itself).
> - Low TTL = changes spread fast but more queries; high TTL = better caching but slow updates (why DNS changes "take time to propagate").
> - **DNS spoofing / cache poisoning:** injecting fake answers into a resolver's cache. Defence: **DNSSEC** (cryptographically signed records). **DoH / DoT** (DNS over HTTPS / TLS) encrypt queries for privacy.
> - DNS is also used for **load balancing** (returning different IPs, round robin / geo-DNS).

## HTTP (HyperText Transfer Protocol)
Request–response protocol for the web over TCP (port 80), or TLS (HTTPS, port 443).
- **Stateless:** the server keeps no memory of previous requests. State is added with cookies, sessions, or tokens.
- **Request** = method + URL + version, headers, optional body. **Response** = version + status code, headers, body.

### Methods

| Method | Purpose | Safe? | Idempotent? | Body? |
|---|---|---|---|---|
| GET | Read a resource | ✅ | ✅ | No (by convention) |
| HEAD | GET without body (headers only) | ✅ | ✅ | No |
| POST | Create / submit data / trigger processing | ❌ | ❌ | Yes |
| PUT | Replace a resource entirely (or create at a known URL) | ❌ | ✅ | Yes |
| PATCH | Partially update a resource | ❌ | ❌ (not guaranteed) | Yes |
| DELETE | Remove a resource | ❌ | ✅ | Optional |
| OPTIONS | Ask which methods are allowed (CORS preflight) | ✅ | ✅ | No |

**Safe** = doesn't change server state. **Idempotent** = doing it N times has the same effect as doing it once.

> [!WARNING]
> **Tricky — GET vs POST:**
> - GET sends parameters in the **URL** (visible, logged, bookmarkable, cached, length-limited); POST sends them in the **body**.
> - POST is **not** secure just because data isn't in the URL — without HTTPS it's plain text either way.
> - **PUT vs POST:** PUT to `/users/42` twice leaves one user 42 (idempotent); POST to `/users` twice creates two users.
> - **DELETE is idempotent** even though the second call returns 404 — the server **state** is the same after both.

### Status codes

| Class | Meaning | Common codes |
|---|---|---|
| 1xx | Informational | 100 Continue, 101 Switching Protocols (WebSocket upgrade) |
| 2xx | Success | **200 OK**, **201 Created**, 202 Accepted, **204 No Content** |
| 3xx | Redirection | **301 Moved Permanently**, **302 Found** (temporary), **304 Not Modified** (use cache), 307/308 (keep method) |
| 4xx | Client error | **400 Bad Request**, **401 Unauthorized**, **403 Forbidden**, **404 Not Found**, 405 Method Not Allowed, 409 Conflict, **429 Too Many Requests** |
| 5xx | Server error | **500 Internal Server Error**, **502 Bad Gateway**, **503 Service Unavailable**, **504 Gateway Timeout** |

> [!WARNING]
> **Tricky:** **401** = not authenticated ("who are you? log in"). **403** = authenticated but not allowed ("I know who you are, and no"). **502** = the proxy got an invalid response from upstream; **504** = the proxy got **no** response in time.

### Useful headers
`Host` (required in HTTP/1.1 — enables virtual hosting), `Content-Type`, `Content-Length`, `Authorization`, `Cookie` / `Set-Cookie`, `Cache-Control`, `ETag` / `If-None-Match` (conditional requests → 304), `User-Agent`, `Accept`, `Connection: keep-alive`, `Location` (with 3xx).

## State: cookies, sessions, tokens (+)
- **Cookie:** small key–value data the server sets via `Set-Cookie`; the browser sends it back on every request to that domain. Attributes: `Expires/Max-Age`, `HttpOnly` (no JavaScript access → XSS protection), `Secure` (HTTPS only), `SameSite` (CSRF protection), `Domain`, `Path`.
- **Session:** server stores the user's state and gives the browser only a **session ID** (in a cookie). Stateful on the server; easy to revoke.
- **Token (e.g. JWT):** the server issues a **signed** token containing claims; the client sends it (usually in `Authorization: Bearer`). The server verifies the signature without storing anything → stateless and scales well, but hard to revoke before expiry.

## HTTP versions

| Version | Key features | Problem left |
|---|---|---|
| **HTTP/1.0** | New TCP connection **per request** | Slow: handshake for every object |
| **HTTP/1.1** | **Persistent connections** (keep-alive, default), **pipelining**, `Host` header, chunked transfer | Responses must return in order → **head-of-line blocking**; browsers open ~6 parallel connections |
| **HTTP/2** | **Binary framing**, **multiplexing** many streams over one TCP connection, **header compression (HPACK)**, stream prioritisation, server push (now deprecated) | TCP-level head-of-line blocking: one lost packet stalls all streams |
| **HTTP/3** | Runs over **QUIC (UDP)**; per-stream loss recovery (no cross-stream HOL blocking), TLS 1.3 built in, **0-RTT/1-RTT** setup, **connection migration** (survives Wi-Fi → mobile IP change via connection IDs) | Some networks block/throttle UDP |

## HTTPS & TLS
**HTTPS = HTTP over TLS** (SSL is the deprecated predecessor). Provides **confidentiality** (encryption), **integrity** (MAC/AEAD), and **authentication** (certificates).

### Symmetric vs asymmetric encryption

| | Symmetric | Asymmetric |
|---|---|---|
| Keys | One shared secret key | Public key + private key pair |
| Speed | Fast | Slow (~1000× slower) |
| Problem | How to share the key securely? | Too slow for bulk data |
| Examples | AES, ChaCha20 | RSA, ECC, Diffie-Hellman (key exchange) |

**TLS combines both:** asymmetric crypto (or Diffie-Hellman) to agree on a secret, then fast **symmetric** encryption for the actual data.

### TLS handshake (simplified TLS 1.2 vs 1.3)
1. **ClientHello:** supported TLS versions, cipher suites, a random number (TLS 1.3: also key-share values).
2. **ServerHello + Certificate:** chosen cipher, server random, the server's **certificate** (its public key signed by a **Certificate Authority**).
3. **Client verifies the certificate** — trusted CA chain, domain name matches, not expired/revoked.
4. **Key exchange:** both sides derive the same **session keys** (ECDHE — ephemeral Diffie-Hellman — gives **forward secrecy**: stealing the server's private key later can't decrypt past sessions).
5. **Finished** messages; encrypted application data flows.

- **TLS 1.2:** 2 round trips before data. **TLS 1.3:** **1 RTT**, with **0-RTT** resumption for returning clients; removed weak ciphers and RSA key exchange.
- **Certificate chain:** server cert → intermediate CA → root CA (pre-installed in the OS/browser trust store).

> [!WARNING]
> **Tricky:**
> - HTTPS encrypts the URL path, headers, and body — but the **domain name is still visible** (DNS query and the TLS **SNI** field, unless ECH is used), and so are the IPs.
> - **Hashing ≠ encryption:** hashing is one-way (integrity, passwords); encryption is reversible with a key.
> - A **digital signature** = hash of the message encrypted with the sender's **private** key; anyone verifies with the **public** key → authentication + integrity + non-repudiation.
> - Encrypt for confidentiality with the **receiver's public** key; sign with the **sender's private** key.

## Email protocols

| Protocol | Role | Port | Notes |
|---|---|---|---|
| **SMTP** | **Send** / push mail (client → server, server → server) | 25 (relay), 587 (submission), 465 (SMTPS) | Push protocol |
| **POP3** | **Retrieve** — download to one device, usually delete from server | 110 (995 secure) | Simple, offline, poor multi-device |
| **IMAP** | **Retrieve** — mail stays on server, synced across devices, folders | 143 (993 secure) | Default today |

> [!WARNING]
> **Tricky:** SMTP only sends; you **can't read** mail with it. Webmail (Gmail in a browser) uses **HTTP** between browser and server, and SMTP between mail servers. MX records tell SMTP where to deliver mail for a domain.

## FTP, SSH, Telnet
- **FTP:** file transfer using **two TCP connections** — **control** (port 21, commands, stays open all session, "out-of-band control") and **data** (a new connection per file).
  - **Active mode:** server opens the data connection **from port 20 to the client** → blocked by client firewalls/NAT.
  - **Passive mode:** client opens the data connection to a port the server announces → firewall-friendly, the default today.
  - Plain FTP sends credentials in **clear text** → use **SFTP** (over SSH) or **FTPS** (over TLS).
- **TFTP:** trivial FTP over UDP port 69, no authentication (network booting, firmware).
- **Telnet (23):** remote terminal, everything in **plain text**. **SSH (22):** encrypted remote login, tunnelling/port forwarding, key-based authentication. SCP/SFTP run over SSH.

## Real-time communication (+)

| Technique | How | Use |
|---|---|---|
| **Short polling** | Client asks repeatedly on a timer | Simple, wasteful |
| **Long polling** | Server holds the request open until data is available, then client re-requests | Near-real-time over plain HTTP |
| **Server-Sent Events (SSE)** | One long-lived HTTP response streaming server → client | Live feeds, notifications (one-way) |
| **WebSocket** | HTTP `Upgrade` (101) to a persistent **full-duplex** TCP channel | Chat, multiplayer games, live collaboration |

## REST basics (+)
Architectural style for web APIs: resources identified by URLs, standard HTTP methods for actions, **stateless** requests, representations (usually JSON), cacheable responses. Other styles: **GraphQL** (client specifies exactly the data it needs, one endpoint), **gRPC** (binary Protocol Buffers over HTTP/2, fast service-to-service calls).

---

## ⭐ What happens when you type `https://www.google.com` and press Enter?

Tell it layer by layer — and go as deep as the interviewer allows.

1. **URL parsing:** browser parses scheme (`https` → port 443), host, and path. If it's not a valid URL, it becomes a search query. **HSTS** list may force HTTPS.
2. **DNS resolution:** browser cache → OS cache / hosts file → resolver → (root → TLD → authoritative, iteratively) → IP address. Usually over **UDP 53**.
3. **Find the next hop:** is the IP on my subnet? No → send to the **default gateway**. Use **ARP** to get the gateway's **MAC** address (if not cached).
4. **TCP 3-way handshake** with the server's IP on port 443 (SYN, SYN-ACK, ACK). (With HTTP/3 this is a QUIC handshake over UDP instead.)
5. **TLS handshake:** negotiate cipher, server sends its **certificate**, browser validates it, both derive session keys.
6. **HTTP request:** `GET / HTTP/2` with headers (`Host`, cookies, `User-Agent`, `Accept`...), encrypted by TLS.
7. **Travel through the network:** segment → IP packet → Ethernet/Wi-Fi frame; **NAT** at the home router rewrites the source IP/port; routers forward hop by hop using **longest prefix match**, decrementing TTL; MAC addresses change at every hop.
8. **Server side:** hits a **load balancer / CDN edge** → web server → application servers → possibly caches and databases → builds the response.
9. **HTTP response:** status (200 / 301 redirect / 304), headers (`Content-Type`, `Cache-Control`, `Set-Cookie`), HTML body.
10. **Rendering:** browser parses HTML → DOM, CSS → CSSOM, runs JavaScript, fetches sub-resources (CSS, JS, images — more requests, often multiplexed over the same connection), lays out and paints the page.
11. **Connection reuse / close:** connection stays open (keep-alive) for further requests; eventually closed with FIN (or kept by the browser's pool).

> [!WARNING]
> **Tricky follow-ups:** "What if DNS returns nothing?" (NXDOMAIN → browser error) · "Where does caching happen?" (browser, OS, DNS resolver, CDN, reverse proxy, server) · "Which protocols use UDP here?" (DNS, QUIC) · "Where does ARP come in?" (to reach the gateway, not google.com) · "Why is the first visit slower?" (DNS + TCP + TLS round trips, cold caches).
