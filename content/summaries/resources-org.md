---
title: resources.org — Reference Learnings Collection
type: summary
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/resources.org
tags: [reference, mental-models, emacs, systems, learnings]
---

# resources.org — Reference Learnings Collection

A heterogeneous collection of learnings, tips, and notes that don't belong to a specific project or area. 2020–2025.

## Key topics

### Mental Models

**The Map Is Not the Territory** — the description of a thing is not the thing itself. Maps are necessary abstractions but dangerous when mistaken for reality. Key applications:
- Education systems: designed for a past job landscape, now outdated.
- News/books: someone else's abstraction of a complex reality — always ask what the cartographer's perspective was.
- Three principles: (1) Reality is the ultimate update; (2) Consider the cartographer; (3) Maps can influence territories.
- Related concept: *Tragedy of the Commons* — individuals rationally overuse shared resources, producing collective harm.

### Emacs / Doom Emacs tips

- `M-o` in ivy to accept input without auto-completing to the first suggestion.
- `(format +onsave)` breaks XML formatting — disable if editing XML.
- `(global-auto-revert-mode t)` to auto-reload buffers when files change on disk (essential for org-roam + Orgzly sync).
- `M-S-RET` on an empty list item creates a checkbox.

### Synology / Homelab

- NFS service in Synology creates an open port; disable when not in active use.
- `nmap -Pn <ip>` to scan open ports.

### Astrophysics

- Sunset illusion: sunlight refracts through atmosphere; when you see the sun "set," it is already below the horizon. Lasts 5–10 minutes.

### Electrical (home improvement)

- Sockets have 3 terminals: Line (hot, always live), Load (to appliance, live only when on), Neutral (returns unused current).
- Ground (bare copper) redirects excess current safely.

## Key concepts referenced

[[Systems Thinking]] (map/territory) · [[NixOS]] (Synology/homelab adjacent)
