+++
date = '2026-10-06T07:47:10Z'
draft = false
title = 'Tree-sitter Never Slowed Down. My Config Did.'
+++

---
I blamed Tree-sitter for months. Open a file with a few thousand lines, and Neovim would stutter on every keystroke. But the parser wasn't the problem. Incremental parsing did its job. What hurt was everything I'd bolted on top: highlight queries with expensive predicates, context plugins walking the tree on every cursor move, and injections spinning up nested parsers. Tree-sitter never slowed down. My config did.

{{< tableofcontents >}}

---
## The symptom

The pattern was consistent: small files felt great, big ones felt broken. In a file with 5,000+ lines, scrolling lagged, typing lagged, and sometimes the editor hung for a second after a paste. Since Tree-sitter was the newest thing in my setup, it got the blame.

---
## Narrowing it down

I stopped guessing and disabled things one at a time on a large file. Turning off Tree-sitter highlighting entirely made most of the lag vanish. Turning it back on while disabling my extras (treesitter-context, rainbow brackets, incremental selection) made most of the lag vanish again. With only highlighting enabled, a little lag remained, which pointed at query cost scaling with the number of nodes. So the parser wasn't the bottleneck. It was the work done after each parse.

---
## The real culprits

Highlight queries were part of it. Predicates like `#match?` and `#any-of?` often run in Lua rather than C, and they run across every captured node, so more lines means more work. Injections added to it: Markdown, HTML, and Vue files spawn nested parsers, each with its own parse and query pass. Tree-walking plugins were the biggest factor. Context, rainbow, and folding each traverse the tree on cursor moves and edits, and while each is cheap alone, together they're painful. Underneath all of it is sheer node count, since a file with thousands of lines produces a huge tree and even incremental updates touch a lot of it.

---
## The fix: stop Tree-sitter per buffer

Instead of removing Tree-sitter, I made it step aside when a buffer gets big. I put the logic in a small utility module so every filetype can reuse it:

```lua
-- lua/utils/treesitter.lua

---@class TreeSitterUtilsOpts
---@field max_lines integer

---@class TreeSitterUtils
---@field max_lines integer
---@field disable_highlighting_for_large_buffers fun(bufnr?: integer, opts?: TreeSitterUtilsOpts)
local M = {}

M.max_lines = 500

---Disable Tree-sitter highlighting for large buffers.
---@param bufnr? integer
---@param opts? TreeSitterUtilsOpts
M.disable_highlighting_for_large_buffers = function(bufnr, opts)
	opts = opts or {}
	bufnr = bufnr or vim.api.nvim_get_current_buf()
	local max_lines = opts.max_lines or M.max_lines

	if vim.api.nvim_buf_line_count(bufnr) >= max_lines then
		vim.treesitter.stop(bufnr)
	end
end

return M
```

Then I call it from `after/ftplugin`, right next to the other per-filetype settings:

```lua
-- after/ftplugin/typescript.lua

local set = vim.opt_local
set.tabstop = 2
set.softtabstop = 2
set.shiftwidth = 2

require('utils.treesitter').disable_highlighting_for_large_buffers()
```

Using `after/ftplugin` matters here. It runs after the default filetype setup, so Tree-sitter has already started by the time I stop it. It also keeps the decision per filetype: I can use a higher limit for languages with cheap queries, or pass `{ max_lines = 2000 }` for a specific one. The tradeoff is that you need the one-line call in each filetype file you care about.

Also, apply the same thinking to your extras. Context, rainbow, and similar plugins often cost more than the highlighting itself, so gate them on the same line count.

---
## What you give up

When Tree-sitter stops, Neovim falls back to regex-based Vim syntax. You lose context-aware colors, but in a file that long you're mostly searching and jumping around, and responsiveness matters more than perfect colors.

---
## Takeaway

Measure before you blame the tool. Tree-sitter is fast, but a fast parser doesn't make a fast editor. Everything you stack on top of the tree has a cost, and large files are where that cost shows up. A line-count check in `after/ftplugin` is a few lines of config and gets you the best of both: accurate highlighting where it helps, speed where it matters.

{{< nextprev >}}
