---
title: Jujutsu
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20250115171906-jujutsu.org
tags: [tools, version-control, git, developer-tools]
---

# Jujutsu (jj)

A distributed version control system designed to solve many of Git's friction points. Uses a different paradigm and is widely praised by those who've adopted it as a dramatically more pleasant experience than Git.

## Key differences from Git

Jujutsu rethinks several Git concepts that cause common headaches:
- The working copy is always a commit (no ambiguous "dirty state").
- Rebasing, amending, and conflict resolution are first-class operations with a cleaner mental model.
- The operation log records every action, making undo trivial.

*Details are a stub — expand after reading the tutorials below.*

## Resources

- [jj init — Chris Krycho's introduction essay](https://v5.chriskrycho.com/essays/jj-init/)
- [Steve Klabnik's Jujutsu Tutorial](https://steveklabnik.github.io/jujutsu-tutorial/)
- [Jujutsu Docs](https://jj-vcs.github.io/jj/)
- [jj config reference](https://github.com/jj-vcs/jj/blob/main/docs/config.md)
- [Starship prompt for jj](https://github.com/jj-vcs/jj/wiki/Starship)
