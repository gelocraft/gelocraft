+++
date = '2026-09-28T02:36:52Z'
draft = false
title = 'Pass It Through and Keep Your Secrets Secret'
+++

---
Every request your users send has to travel through several layers of infrastructure before it reaches your application. Somewhere along that path, a load balancer or proxy has to decide whether to simply forward the traffic or to open it up and read it. That decision determines who can see your most sensitive data, and TLS passthrough is the approach that keeps it visible to no one but the two parties involved.

{{< tableofcontents >}}

---
## The Problem with Terminating TLS

In most setups, a load balancer or reverse proxy sits in front of your application. The usual approach is TLS termination. The proxy decrypts incoming traffic, inspects it, routes it, and then sends it to the backend either in plain text or re-encrypted with a new connection.

That works fine for many cases, but it means the proxy holds your private key and sees every byte of your traffic. It's like handing your diary to a mailman so he can "route it more efficiently." If that proxy is compromised, misconfigured, or run by someone you don't fully trust, your secrets are exposed.

---
## What TLS Passthrough Does

With TLS passthrough, the proxy never decrypts anything. It receives the encrypted stream, reads just enough to know where to send it, and forwards it untouched to the backend. The backend server completes the TLS handshake with the client directly. Think of the proxy as a very polite bouncer who checks which club you're headed to, points you to the door, and has absolutely no interest in what's in your pockets.

The routing decision usually relies on SNI (Server Name Indication), a field in the TLS handshake that tells the proxy which hostname the client wants. The hostname is visible, but the actual content of the conversation is not. The proxy knows where you're going, just not why you're going there, which is honestly how most of us prefer our relationships with infrastructure.

---
## The Trade-Offs

TLS passthrough is not free. Because the proxy can't read the traffic, you lose a lot of Layer 7 features. Your proxy is now working blindfolded, and it can't do much beyond pointing in a general direction.

- No path-based routing, header rewriting, or cookie-based session stickiness
- No WAF inspection or content-based filtering at the proxy
- Limited visibility into requests for logging and debugging (good luck asking the proxy what happened, it genuinely doesn't know)
- Each backend has to manage its own certificates

If you need those features, TLS termination or re-encryption is the better fit. Many teams use both, with TLS passthrough for sensitive services and TLS termination for everything else.

---
## TLS Passthrough or TLS Termination, Which One to Choose?

Choose **TLS passthrough** when:

- **The backend must own the encryption.** Your application should be the only place where traffic is decrypted.
- **You need mutual TLS.** Your backend has to validate client certificates directly, so the original handshake must reach it intact.
- **You handle non-HTTP traffic.** Databases, message brokers, and internal services running TLS work best when the proxy just passes the bytes along.
- **You don't trust the middle.** If the proxy runs on shared or third-party infrastructure, keeping it blind to the content removes a major risk.

Choose **TLS termination** when:

- **You need smart routing.** Path-based routing, header rewriting, and cookie-based stickiness all require the proxy to read the request.
- **You want inspection and filtering.** WAF rules and content-based security checks only work on decrypted traffic.
- **You want centralized certificates.** Managing certificates in one place is easier than handling them on every backend.
- **You need detailed logs and visibility.** Debugging and analytics are much simpler when the proxy can see full requests.

If end-to-end encryption is your priority, go with TLS passthrough. If you need Layer 7 features, go with TLS termination. Many teams use both, with TLS passthrough for sensitive services and TLS termination for everything else.

---
## Why TLS Passthrough Is the Best Option for End to End Encryption

If the goal is genuine end-to-end encryption, TLS passthrough is the only approach that fully delivers it. Every other option breaks the promise somewhere along the way.

Think about what "end to end" actually means. It means that only the two parties in the conversation can read the data, and nobody in the middle can. TLS termination fails this test by definition, because the proxy decrypts the traffic and becomes a third party that sees everything. That's not end to end, that's end to middle to end, which is a much less catchy marketing phrase.

Re-encryption does not fix this. It just wraps the data in a new layer after the proxy has already read it. That's like opening someone's letter, reading it, and then sealing it in a fresh envelope while saying "nothing to see here." The plain text still exists in the proxy's memory, and anyone who compromises that machine gets everything.

TLS passthrough removes that weak point entirely. There is no decryption in the middle, so there is nothing to steal in the middle. A breached load balancer, a rogue administrator, or a misconfigured logging rule cannot expose what the proxy never had access to. The private key lives only on the backend, which shrinks your attack surface to a single, well-defended place instead of spreading it across every device in the path.

It also keeps the trust model honest. With TLS passthrough, the client is verifying the identity of the real application server, not an intermediary standing in its place. Certificate validation, client authentication, and cipher negotiation all happen between the two real endpoints, exactly as TLS was designed to work.

Other approaches ask you to trust the infrastructure between you and your users. TLS passthrough asks you to trust nothing in between, and that is what end-to-end encryption is supposed to mean. Your secrets stay secret, and your proxy gets to say, with total honesty, "I have no idea what's going on in there."

{{< nextprev >}}
