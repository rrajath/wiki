---
title: Easy Plug-and-Play
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20220731223202-easy_plug_and_play.org
tags: [design, portability, software]
---

# Easy Plug-and-Play

A design principle: build components that are **self-contained and portable** — they can be picked up and dropped into a new context without modification.

## Core idea

When breaking a problem into components, prefer formats and interfaces that are not tied to one specific implementation. Portability lets you swap out the underlying tool without losing your work.

## Example

Writing blog content in Markdown: most static site generators support the same Markdown + frontmatter format, so migrating from one generator to another is a matter of copying files, not rewriting content. The content is decoupled from the renderer.

## Relation to system design

Plug-and-play components reduce lock-in and switching costs. The principle generalizes: any time you can define a stable interface between layers (content vs presentation, data vs storage engine), you gain the freedom to evolve each layer independently.

See also: [[Workflow]] — composable productivity improvements follow the same logic.
