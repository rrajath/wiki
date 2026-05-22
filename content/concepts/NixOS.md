---
title: NixOS
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20250108153813-nixos.org
  - raw/notes/20250108154131-home_manager.org
  - raw/notes/20250108154324-nix_resources.org
tags: [nix, nixos, linux, dotfiles, tools]
---

# NixOS

A Linux operating system (and package manager) where the entire system — OS, drivers, apps, packages, and settings — is declared in configuration files. Reproducible, rollback-capable, and fully declarative.

## Why it's interesting

Traditional system configuration is imperative: install this, then configure that, remember the manual steps. NixOS is declarative: describe the desired system state in a `.nix` file and Nix makes it so. Setting up a new machine becomes "copy the config, run nix rebuild."

## Nix on macOS (nix-darwin)

Nix can be installed on macOS via `nix-darwin`, enabling the same declarative package management without switching OS. This makes it practical for macOS users who want reproducible system configs without running Linux.

## Home Manager

An addon that extends Nix into per-user configuration: packages, dotfiles, and program settings (Emacs, Helix, Vim, Nushell, Fish, Zsh, Aerospace, etc.) are all declared in Home Manager config.

Benefits:
- Everything in one place — no scattered dotfiles repo.
- A new machine install automatically applies all program configuration — no manual setup.
- Configs are version-controlled, reproducible, and rollback-capable.

This replaces (or supplements) a traditional dotfiles folder with literate configuration.

## Key resources

- [NixOS & Flakes Book](https://nixos-and-flakes.thiscute.world/)
- [Nix Pills](https://nixos.org/guides/nix-pills/) — foundational learning
- [Home Manager Option Search](https://rycee.gitlab.io/home-manager/options.xhtml)
- [NixOS Package Search](https://search.nixos.org/packages)
- [MyNixOS](https://mynixos.com/)
- [Vimjoyer on YouTube](https://youtu.be/FcC2dzecovw)
