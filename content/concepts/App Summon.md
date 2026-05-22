---
title: App Summon
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20220727000001-app_summon.org
tags: [productivity, keyboard, macos]
---

# App Summon

A keyboard technique that replaces Cmd+Tab app switching with a dedicated hotkey per application — directly bringing a specific app to the foreground in one keystroke.

## The problem with Cmd+Tab

Cmd+Tab is pressed hundreds of times a day. Each press requires scanning a list and navigating to the right icon — a visual search task repeated constantly. The cognitive overhead is small per press but large in aggregate.

## The technique

Assign a unique hotkey (typically a Meh or Hyper key combination — e.g., `Hyper+B` for browser, `Hyper+E` for editor) to summon each app directly. No list scanning. One chord = specific app in foreground.

Popularized by Ben Vallack (YouTube). Tools: Karabiner-Elements (macOS) for Hyper key setup + a launcher like Hammerspoon or Raycast for the bindings.

## As a workflow building block

App Summon is a [[Productivity Improvement]] on its own. Its real power is as a component in a [[Workflow]]:
- Summon + DND toggle + full screen = instant deep-work context.
- Summon + window layout = instantly reproducible workspace.
