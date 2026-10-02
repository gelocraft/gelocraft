+++
date = '2026-10-02T03:21:30Z'
draft = false
title = 'Byte Order for People Who Just Want Their Data to Parse'
+++

---
You send the number 1 from one computer to another, and the receiver tells you it got 16,777,216. Nothing got lost on the way and the connection is fine. The two machines just disagree about which byte comes first.

{{< tableofcontents >}}

---
## What is byte order?

Big numbers don't fit in one byte, so computers spread them over several. Take the number `0x12345678`. It needs four bytes, and there are two ways to line them up in memory.

Big-endian puts the biggest end first, so you get `12 34 56 78`. Little-endian does the opposite and gives you `78 56 34 12`.

You've probably mixed up a date before. Someone writes 05/06, and you can't tell if it's May 6th or June 5th. Nothing is wrong with the numbers. You just need to know which order the writer used.

---
## The wild early days of byte order

There was no industry standard for byte order when the first computers were built, so designers chose whatever made their hardware simpler or faster. Nobody thought of it as a problem, because most machines never had to share numbers with each other.

Over time, two camps formed. Motorola and IBM leaned toward big-endian, which matched the way people write numbers. Intel and DEC leaned toward little-endian, which suited the way their processors handled arithmetic.

Neither side won. Today x86 and most ARM chips run little-endian, while some older and specialized systems stay big-endian. The trouble started when these machines had to talk to each other and found they didn't agree.

---
## Why networks care

Two computers talking to each other can have totally different CPUs. If each one sent numbers in its own favorite order, they'd misread each other all the time.

So networking settled on one shared order, which is big-endian. You'll hear it called network byte order.

---
## The rule for sending and receiving bytes

When sending bytes, convert your numbers to big-endian before they go onto the network.

When receiving bytes, convert them from big-endian to your own machine's byte order, known as "host byte order".

If your machine is already big-endian, the conversion changes nothing. If it's little-endian, the bytes get swapped. You write the same code either way, so it behaves the same on every machine.

If a small number suddenly shows up huge, like 1 turning into 16,777,216, one side probably skipped the conversion.

Byte order can seem intimidating at first, but it's just an agreement about which end of a number goes first. Convert on the way out, convert back on the way in, and your numbers will arrive exactly as you sent them.

{{< nextprev >}}
