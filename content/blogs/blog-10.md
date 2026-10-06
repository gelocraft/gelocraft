+++
date = '2026-10-06T07:14:50Z'
draft = false
title = 'I Quit LSP and Went Back to ctags'
+++
---
I'm done using LSP on large codebases. It kept crashing, hanging, and eating gigabytes of RAM, and after every branch switch it spent minutes re-indexing while my editor crawled. So I turned it off and went back to ctags.

To be fair, LSP is great on small and medium projects. Accurate go-to-definition, reliable rename, instant diagnostics, and completion that understands types make it feel like magic when the codebase fits comfortably in memory. The problem is that all of that comes from deep semantic analysis: building type information, resolving imports, and tracking references across the whole project. That work scales badly. Past a certain size, the server spends more time catching up than helping you.

ctags goes the opposite direction. It doesn't try to understand your code, it just scans files and writes a flat index of where symbols are defined. That simplicity is exactly why it scales. It can index millions of lines in seconds rather than minutes, and the index is just a plain file, so there's no resident process eating RAM and nothing to crash. It works on practically any language, even over SSH in a bare terminal, and it behaves the same on a 10k-line repo as it does on a 10-million-line one.

What did I give up? Less than I expected. Renaming is the one people worry about, but I just search with grep or rg, load the results into the quickfix list, and run a substitution across it with `:cfdo %s/old/new/gc`. I review every match as I go, which honestly feels safer than trusting a blind automated rename. Ambiguous names sometimes give me several tag candidates to pick from, and I don't get type-aware references, but in a huge codebase, an instant "good enough" jump beats a precise answer that arrives late, or never.

LSP is still my pick for small and medium projects. For large codebases, I'm done with it.

{{< nextprev >}}
