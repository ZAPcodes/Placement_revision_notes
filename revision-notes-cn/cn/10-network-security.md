# 10 — Network Security ★★

## CIA triad
- **Confidentiality:** only authorised parties can read data → encryption, access control.
- **Integrity:** data isn't altered undetected → hashes, MACs, digital signatures.
- **Availability:** systems stay usable → redundancy, DDoS protection, backups.

Plus **authentication** (who are you?), **authorisation** (what may you do?), and **non-repudiation** (you can't deny sending it → digital signatures).

## Cryptography basics

| Concept | Purpose | Examples |
|---|---|---|
| **Symmetric encryption** | Confidentiality, fast, one shared key | AES, ChaCha20 (DES/3DES obsolete) |
| **Asymmetric encryption** | Key exchange, signatures; public/private pair | RSA, ECC |
| **Key exchange** | Agree on a shared secret over an insecure channel | Diffie-Hellman, ECDHE |
| **Hashing** | Integrity; fixed-size, one-way fingerprint | SHA-256 (MD5, SHA-1 broken) |
| **MAC / HMAC** | Integrity + authenticity with a shared key | HMAC-SHA256 |
| **Digital signature** | Integrity + authentication + non-repudiation | RSA/ECDSA signatures |
| **Certificate** | Binds a public key to an identity, signed by a CA | X.509 certs in TLS |

> [!WARNING]
> **Tricky:**
> - **Encoding** (Base64) is not encryption — anyone can decode it. **Hashing** can't be reversed. **Encryption** is reversible with the key.
> - Passwords should be stored as **salted, slow hashes** (bcrypt, Argon2), never encrypted and never plain SHA-256.
> - **n users** need **n(n−1)/2** keys with symmetric crypto, but only **2n** keys (one pair each) with asymmetric.
> - Plain Diffie-Hellman is vulnerable to man-in-the-middle unless the exchange is **authenticated** (that's what certificates do in TLS).

## Common attacks

| Attack | What happens | Defence |
|---|---|---|
| **DoS / DDoS** | Flood a target with traffic/requests to exhaust bandwidth, CPU, or connections; DDoS uses a botnet of many machines | Rate limiting, CDN/scrubbing services, filtering, autoscaling |
| **SYN flood** | Many half-open TCP connections fill the server's backlog | SYN cookies, backlog tuning |
| **Man-in-the-middle (MITM)** | Attacker secretly relays/alters traffic between two parties | TLS with certificate validation, HSTS, VPNs |
| **ARP spoofing** | Fake ARP replies map the attacker's MAC to the gateway's IP → MITM on the LAN | Dynamic ARP inspection, static ARP entries, encryption |
| **DNS spoofing / cache poisoning** | Fake DNS answers send users to malicious IPs | DNSSEC, randomised query IDs and ports |
| **IP spoofing** | Forged source IP (used in DDoS reflection) | Ingress/egress filtering |
| **Phishing** | Tricking users into giving credentials | User awareness, MFA, email filtering (SPF/DKIM/DMARC) |
| **Session hijacking** | Stealing a session cookie/token | HTTPS, `HttpOnly` + `Secure` cookies, short sessions |
| **Replay attack** | Re-sending captured valid messages | Nonces, timestamps, sequence numbers |
| **Packet sniffing** | Capturing unencrypted traffic | Encryption (TLS, SSH, VPN) |

### Web attacks commonly asked in CN rounds (+)
- **SQL injection:** user input is concatenated into SQL, changing the query (`' OR '1'='1`). **Fix:** parameterised queries / prepared statements.
- **XSS (Cross-Site Scripting):** attacker's script is injected into a page and runs in other users' browsers (stealing cookies). **Fix:** output encoding/escaping, Content Security Policy, `HttpOnly` cookies.
- **CSRF (Cross-Site Request Forgery):** a malicious site makes the victim's browser send an authenticated request to another site (cookies attach automatically). **Fix:** CSRF tokens, `SameSite` cookies.

> [!WARNING]
> **Tricky — XSS vs CSRF:** XSS **runs the attacker's code** in the trusted site's context (exploits the user's trust in the site). CSRF **makes the user's browser send a request** they didn't intend (exploits the site's trust in the user's browser). XSS can defeat CSRF tokens.

## Security protocols by layer

| Layer | Protocol |
|---|---|
| Application | HTTPS, SSH, S/MIME, PGP, DNSSEC |
| Transport | **TLS** (also DTLS over UDP) |
| Network | **IPSec** (AH for integrity/authentication, ESP for encryption; transport vs tunnel mode) |
| Data link | WPA2/WPA3 (Wi-Fi), 802.1X, MACsec |

> [!WARNING]
> **Tricky — IPSec vs TLS:** IPSec secures **all** IP traffic between two hosts or networks, transparently to applications (used for site-to-site VPNs). TLS secures **one application's connection** and the app must use it (HTTPS). IPSec **transport mode** encrypts only the payload; **tunnel mode** encrypts the whole original packet and adds a new IP header.

## Authentication essentials (+)
- **MFA:** something you know (password) + something you have (phone/OTP/hardware key) + something you are (biometric).
- **OAuth 2.0:** delegated **authorisation** ("let this app access my Google Drive") via access tokens. **OpenID Connect** adds **authentication** (identity) on top. **SSO:** one login for many services.
