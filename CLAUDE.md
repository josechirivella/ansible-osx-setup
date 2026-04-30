# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an Ansible-based Mac (and Ubuntu) setup automation repository. It provisions a machine from scratch using playbooks, roles, and templates.

## Running the Playbooks

**Initial setup (fresh Mac):**
```sh
bin/bootstrap
```
This installs Xcode CLI tools, Ansible, fetches required Galaxy roles/collections, then runs the main playbook.

**Apply changes to an existing machine:**
```sh
bin/apply
```
Equivalent to: `ansible-playbook -i "localhost," -c local ansible_osx.yml --ask-become-pass`

**Ubuntu:**
```sh
ansible-playbook -i "localhost," -c local ansible_ubuntu.yml --ask-become-pass
```

**Install/update Ansible Galaxy dependencies:**
```sh
ansible-galaxy install -r requirements.yml
```

## Architecture

| File/Dir | Purpose |
|---|---|
| `ansible_osx.yml` | Main macOS playbook — single entry point for all Mac provisioning |
| `ansible_ubuntu.yml` | Equivalent playbook for Ubuntu |
| `bin/bootstrap` | One-shot script for a fresh machine (installs Ansible, then runs `ansible_osx.yml`) |
| `bin/apply` | Re-runs `ansible_osx.yml` for incremental updates |
| `requirements.yml` | Ansible Galaxy collections: `community.general` (git) and `geerlingguy.mac` |
| `roles/dnsmasq/` | Optional role to set up dnsmasq as a local DNS resolver |
| `templates/` | Jinja2 templates for dotfiles and config files (`.zshrc`, VSCode settings/keybindings, `.asdfrc`) |
| `tasks/` | Standalone task files (e.g., `docker.yml`, `ubuntu/google-chrome.yml`) |

## Key Design Decisions

- **`ansible_osx.yml` is the source of truth** for what gets installed. To add or remove software, edit that file directly.
- **Homebrew casks** use `community.general.homebrew_cask` with `ignore_errors: yes` — cask failures are non-fatal.
- **mise** manages Node (v24), Rust (latest), and Ruby (latest) as version-managed runtimes, replacing the older nvm/asdf approach.
- **pnpm** is the Node package manager; `@angular/cli` and `typescript` are installed globally.
- Several tasks in `ansible_osx.yml` are commented out (ZSH config template, VSCode config/keybindings templates, VSCode extension installs) — these exist as reference but are not applied.
- The `dnsmasq` role must be started manually with `sudo brew services start dnsmasq` after provisioning.
- The bootstrap process may require multiple terminal restarts due to PATH not being fully set up mid-run — this is a known limitation noted in the README.
