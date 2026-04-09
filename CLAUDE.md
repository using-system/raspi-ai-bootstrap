# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

raspi-ai-bootstrap uses Ansible to bootstrap a Raspberry Pi 5 (Raspberry Pi OS Lite 64-bit / Bookworm) into a self-hosted AI node with Claude Code, MCP tools, Telegram integration, and local LLM inference.

## Commands

```bash
# Syntax check
ansible-playbook playbooks/bootstrap.yml --syntax-check

# Dry-run (no changes applied)
ansible-playbook playbooks/bootstrap.yml --check

# Run the full bootstrap
ansible-playbook playbooks/bootstrap.yml

# Run a specific role only
ansible-playbook playbooks/bootstrap.yml --tags base
ansible-playbook playbooks/bootstrap.yml --tags docker
ansible-playbook playbooks/bootstrap.yml --tags claude-code
```

## Architecture

- **Inventory** (`inventory/hosts.yml`): Single Pi host with static IP
- **Playbook** (`playbooks/bootstrap.yml`): Orchestrates all roles in order
- **Roles**: Each role is self-contained with `defaults/main.yml` for variables and `tasks/main.yml` for logic
  - `base` — OS config: apt upgrade, packages, locale, timezone, hostname, UFW firewall
  - `docker` — Docker Engine CE from official repo + compose plugin
  - `claude-code` — Node.js LTS via NodeSource + `@anthropic-ai/claude-code` via npm

## Conventions

- Target OS: Raspberry Pi OS Lite (Debian Bookworm, arm64)
- Default user: `pi`
- All roles use FQCNs (e.g. `ansible.builtin.apt`, `community.general.ufw`)
- Variables are defined in each role's `defaults/main.yml` and can be overridden via inventory or extra vars
