+++
date = '2026-09-26T10:46:02Z'
draft = false
title = 'One Port, Many Protocols: How ALPN Makes HTTP/2 Possible'
+++

---
Here's a problem you probably never think about: your browser connects to port 443 for HTTPS, but that port doesn't know if you want HTTP/1.1 or HTTP/2. Same endpoint, two completely different wire formats. Somebody has to decide which one to use — before any actual HTTP data gets sent.

That's ALPN's job.

ALPN (or Application-Layer Protocol Negotiation) is a small TLS extension that lets the client say "here's what I support" during the TLS handshake, and the server picks one. For HTTP/2 the identifier is **h2**; for HTTP/1.1 it's **http/1.1**.

{{< tableofcontents >}}

---
## The Problem: One Port, Two Protocols

HTTP/1.1 and HTTP/2 aren't compatible on the wire. HTTP/1.1 is the plain text you're used to:

{{< highlight md "hl_lines=1-2" >}}
GET /index.html HTTP/1.1
Host: example.com
{{< /highlight >}}


HTTP/2 is binary, split into frames and streams. A server can't just guess which one it's looking at based on the connection alone — it needs to know up front.

---
## What ALPN Actually Does

The client lists the protocols it supports (say, **h2** and **http/1.1**) inside the TLS ClientHello, and the server picks whichever one both sides can handle. This all happens before anything application-level gets exchanged, so by the time HTTP data starts flowing, both sides already agree on the format.

One nuance worth knowing: the server can only pick from what the client offers. If a client only sends **http/1.1**, the server can't unilaterally decide to use HTTP/2, even if it supports it.

---
## Why HTTP/2 Doesn't Need Its Own Port

Without ALPN, you'd probably need separate ports — 443 for HTTP/1.1, something else for HTTP/2. Instead, the same port 443 can serve both, and ALPN sorts out which protocol to actually speak based on what the client offers. Older clients that only know HTTP/1.1 still work fine; they just never offer **h2**, so the server falls back automatically. Nothing breaks.

---
## Seeing ALPN in Action (with OpenSSL)

You can watch this negotiation happen yourself:

{{< highlight md "hl_lines=1" >}}
openssl s_client -connect example.com:443 -alpn h2,http/1.1
{{< /highlight >}}

The response tells you which protocol got selected:

{{< highlight md "hl_lines=1" >}}
ALPN protocol: h2
{{< /highlight >}}

or

{{< highlight md "hl_lines=1" >}}
ALPN protocol: http/1.1
{{< /highlight >}}

---
## ALPN and HTTP/3

Same idea, different identifier: **h3**. HTTP/3 runs over QUIC instead of TCP, so the transport is different, but ALPN still handles protocol identification the same way. Good reminder that ALPN was never HTTP/2-specific — it's a general-purpose negotiation mechanism.

---
## Why It Matters for Network Engineers

If you work on reverse proxies, load balancers, or anything doing TLS termination, ALPN is a nice illustration of how cleanly the layers separate: TCP handles transport, TLS handles security, ALPN handles protocol negotiation, and HTTP/1.1 or HTTP/2 takes it from there. A proxy can even negotiate HTTP/2 with the browser while talking HTTP/1.1 to its backend — the two sides don't have to match.

{{< nextprev >}}
