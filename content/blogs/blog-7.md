+++
date = '2026-09-26T11:39:39Z'
draft = false
title = 'Decoding the Journey of a Single HTTP Request'
+++

---
We treat it like magic: type an address, tap enter, and a fully-formed webpage appears almost instantly. But behind that single click sits a surprisingly intricate relay race — a sequence of lookups, handshakes, and negotiations happening in milliseconds, invisible to the person waiting on the other end. Your browser doesn't just "ask" a server for a page. It first has to figure out *where* that server even lives, build a trusted, reliable connection to it, agree on a shared language to speak, and only then does it dare send the actual request. Miss any one of these steps and nothing loads at all. Understanding this chain isn't just trivia — it's the foundation for debugging slow page loads, understanding HTTPS security, and appreciating just how much coordination the internet quietly pulls off every single time you browse. Let's walk through it step by step.

{{< tableofcontents >}}

---
## 1. DNS Resolution

Your browser needs an IP address for the domain. It checks its own cache, then the OS cache, then asks a recursive resolver (often your ISP or something like 8.8.8.8). If that resolver doesn't know, it walks the DNS hierarchy, starting at the root servers, moving to the TLD servers (.com, .org), and finally reaching the domain's authoritative nameserver, which returns the IP.

---
## 2. TCP Handshake
With an IP in hand, your machine opens a TCP connection to the server (usually port 443 for HTTPS). This is the classic three-way handshake:
- Client sends **SYN**
- Server replies **SYN-ACK**
- Client sends **ACK**

Now there's a reliable, ordered connection — but still no encryption yet.

---
## 3. TLS Handshake
For HTTPS, a TLS handshake happens next, layered on top of that TCP connection:
- Client sends a **ClientHello** — supported TLS versions, cipher suites, and a random number
- Server responds with a **ServerHello**, its certificate, and picks a cipher suite
- Both sides derive shared session keys (via a key exchange like ECDHE)
- A **Finished** message confirms both sides can now encrypt/decrypt

Modern TLS 1.3 does this in one round trip instead of two.

---
## 4. SNI (Server Name Indication)
Here's a subtlety: at the TCP layer, the server only knows an IP — but one IP often hosts many domains (shared hosting, CDNs). So during the ClientHello, the browser includes **SNI**, plainly stating which hostname it wants (e.g., `example.com`), so the server can present the *right* certificate before encryption even starts.

---
## 5. ALPN (Application-Layer Protocol Negotiation)
Also tucked into that same ClientHello: **ALPN**. This lets the client and server agree on which application protocol to speak — HTTP/1.1 or HTTP/2 (`h2`) — *during* the TLS handshake, rather than negotiating it afterward. Saves a whole extra round trip.

---
## 6. The HTTP Request Itself
Only now, over this encrypted tunnel, does the actual HTTP request go out:

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: ...
Accept: text/html
```
The server processes it and sends back a response — status line, headers, body — and your browser renders the page.

That's the full journey: DNS finds the address, TCP builds the connection, TLS locks it down while SNI and ALPN quietly settle which certificate and protocol to use, and only then does HTTP finally carry the request that gets you your page.

{{< nextprev >}}
