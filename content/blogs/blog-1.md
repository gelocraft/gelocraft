+++
date = '2026-09-19T04:55:43Z'
draft = false
title = 'Where should I host my website? — VPS, CDN, Serverless'
+++

---
So, you’ve built your first-ever website or frontend app, but you don’t know where to host it? Good news—you’ve come to the right place.

In this article, I’m going to go over the different options available for hosting your website, discuss the downsides of each option, and explain why you might choose one over another.  

{{< tableofcontents >}}

---

## Virtual Private Server (VPS)

For the past few years, a VPS has been the traditional option for hosting a website. You build a website and serve its files over the internet using popular web servers like Apache or Nginx. Even today, as I’m writing this article, lots of company websites on the internet are still running on VPS. And yes, I’ve mentioned VPS multiple times without actually explaining what it is.

VPS stands for **Virtual Private Server**. Basically, it’s a cloud server available for rent that people use to host their websites, run their own VPN with WireGuard, or whatever other use case you can think of—because I’m running out of examples here, as you might have noticed.

A VPS is basically a computer without a graphical user interface that runs 24/7 in the cloud. But not literally _the cloud in the sky_. (And of course, I’m not going to explain what “the cloud” is _like you’re five_. I’m just going to assume you’re a big boy who knows how to search for things on the internet.)

Okay, enough with the explanations. Let’s talk about the maintenance and mental overhead of hosting your website on a VPS.

First, **DNS records**. This is one of the first things you’ll need to configure after buying a domain name for your website. You add an **A record** pointing to the IPv4 address of your VPS at your domain registrar (e.g., Namecheap). This allows people to find your website without having to type the actual IPv4 address into their browser.

Second, the web server (e.g., Nginx) that serves your website over the internet needs to support **HTTPS** (the **S** stands for **secure**). Modern browsers generally warn users about, and may restrict access to, websites that don’t use HTTPS.

Running your website over HTTPS requires you to obtain a TLS certificate for your domain and have it issued by a Certificate Authority (CA), such as Let’s Encrypt. The TLS certificate allows browsers to verify that they’re connecting securely to the server associated with your domain.

And I almost forgot to mention that TLS certificates have an expiration date. You need to renew them periodically to keep your website running over HTTPS. Thankfully, tools like Certbot can automate this process, so you can schedule automatic renewals instead of manually renewing your certificate every time.

To put it all together, the TLS certificate is one of the key pieces that enables HTTPS on your website.

Having only one server for your website also leads to a problem called a **single point of failure**. Basically, if your VPS goes offline or becomes unavailable, no one can access your website until the server comes back online.

And this is where we look at the second option: **CDNs**.



---
## Content Delivery Network (CDN)

If you're building a static website or a SPA, just use a CDN.

Seriously.

You don't need to rent a VPS. You don't need to configure Nginx. You don't need to SSH into a server at 2 AM because your website suddenly decided to stop working. You definitely don't need to spend your weekend learning Linux server administration just to serve an "index.html" file.

A CDN is probably one of the simplest ways to get a website online today.

So, what exactly is a CDN?

CDN stands for Content Delivery Network.

It's a network of servers distributed across different locations around the world. These servers cache and deliver your website's static assets—HTML, CSS, JavaScript, images, fonts, and other files—to users from locations closer to them.

Imagine your website is hosted in the United States, but your visitor is in the Philippines.

Without a CDN, that visitor has to fetch the website directly from the server in the United States.

With a CDN, the visitor can get a cached copy from a nearby edge server instead.

That's the main idea: put your content closer to your users.

### Why I prefer CDN for websites

The biggest advantage is that you don't have to manage a server.

With a VPS, you're responsible for the operating system, security updates, Nginx, firewall rules, TLS certificates, monitoring, backups, and all the other exciting problems that come with owning a server.

With a CDN, most of that is someone else's problem.

You build your website.

You deploy it.

The CDN distributes it.

That's it.

Static websites are basically the perfect workload for this.

If your website is just HTML, CSS, JavaScript, images, and fonts, there's very little reason to rent an entire virtual machine just to serve those files.

### What about SPAs and SSR?

CDNs are also great for Single Page Applications. A production React, Vue, Svelte, Angular, or vanilla JavaScript application usually ends up as a collection of static files, which is exactly what a CDN is good at serving.

But SSR (Server-Side Rendering) is a different story. SSR requires code to run on the server to generate the HTML dynamically.

Traditional CDNs can't do that by themselves, but many modern CDN providers now offer edge computing, allowing you to run certain types of server-side code closer to your users.

Once your application requires long-running processes, persistent connections, heavy computation, or a traditional backend, a VPS or another server-based solution may make more sense.

### What about cost?

For small websites, CDNs can also be extremely cheap, and many providers offer generous free tiers.

You're not paying for an entire computer sitting around 24/7 just to serve a few megabytes of JavaScript.

Of course, always check the provider's bandwidth limits and pricing. Free doesn't mean unlimited.

And last but not least, let's talk about serverless.

---
## Serverless

The name "serverless" is probably one of the most misleading names in software engineering.

There are still servers.

I repeat: **there are still servers.**

Someone didn't just unplug the servers and throw them into the ocean.

Serverless simply means **you don't have to manage the underlying servers yourself**. The provider takes care of provisioning, scaling, and maintaining the infrastructure while you deploy your code on top of it.

Sounds great, right?

Well, I'm going to disagree with the idea that serverless is the best way to host a website.

### Why would you use serverless for a website?

A lot of people use serverless functions to host their frontend or backend because it's convenient. You deploy your application, the platform handles the infrastructure, and you don't have to spend your afternoon configuring Nginx and wondering why your server has suddenly eaten all of its RAM.

For a small application, this can be perfectly reasonable.

But for a simple website, I think you're often introducing unnecessary complexity.

If all you need to serve is some HTML, CSS, JavaScript, images, and maybe a small API, you probably don't need a bunch of serverless functions running behind your website.

A CDN can already handle static files extremely well, and if you need a traditional backend, a small VPS can give you a lot more control for a predictable price.

### Your bill can surprise you

One of the biggest things I don't like about serverless is the **pay-per-use pricing model**.

Instead of paying a fixed amount for a server every month, you're often charged based on things like requests, execution time, memory usage, bandwidth, and other resources.

That sounds great when your application has very little traffic.

But imagine your website suddenly gets a lot of traffic.

Maybe your blog post goes viral.

Maybe someone puts your website on Reddit.

Maybe a bot discovers your API and decides that your `/api/users` endpoint is its new favorite hobby.

Your application can suddenly start generating a lot more requests.

And if you're not paying attention to your usage and billing, your bill can start climbing before you even realize what's happening.

**Your website gets popular and your credit card gets character development.**

This is especially important when you're learning or experimenting. It's very easy to deploy something, forget about it, and assume that because you're not managing a server, there's nothing to worry about.

There is.

Always check the pricing model, usage limits, billing alerts, and spending controls of the platform you're using.

### Cold starts

Serverless functions can also introduce **cold starts**.

When a function hasn't been used for a while, the platform may need to initialize a new execution environment before running your code. Depending on the platform and runtime, this can add latency to the request.

Modern platforms have improved this significantly, but it's still something to consider when you're building latency-sensitive applications.

### Execution limits

Serverless functions also aren't designed to run forever.

They usually have limits on execution time, memory, CPU, request size, and other resources.

That's perfectly fine for short-lived tasks such as handling an API request.

But if your application needs long-running processes, background workers, persistent connections, or heavy computation, a traditional server may be a better fit.

### Vendor lock-in

There's also **vendor lock-in**.

Serverless platforms tend to provide a lot of convenient services around their functions: databases, authentication, storage, queues, cron jobs, edge functions, and more.

The more of these services you use, the more your application starts depending on that particular provider.

Moving a Docker container from one VPS to another can be relatively straightforward.

Moving an application that depends on ten different proprietary serverless services?

Well...

Good luck.

### So, should you never use serverless?

No.

Serverless can be a great choice for the right workload.

If you have an API with unpredictable traffic, short-lived background jobs, webhooks, or other workloads that benefit from automatic scaling, serverless can save you a lot of infrastructure work.

My argument is specifically about **using serverless as the default answer for hosting a website**.

If you're just trying to host a static website, portfolio, blog, or SPA, I'd rather keep things simple.

Use a CDN for static assets.

Use a small VPS if you actually need a traditional server.

Use serverless when you have a workload that benefits from what serverless actually provides.

Don't use it just because someone on Twitter said you can build an entire startup without touching a server.

Because technically, they're right.

You just might end up touching your credit card instead.

{{< nextprev >}}
