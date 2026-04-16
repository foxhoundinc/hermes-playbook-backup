---
name: github-playbook-backup
description: GitHub Repository Setup for Hermes Playbook Backup
category: development
license: MIT
metadata:
  author: cjhermes
  version: "1.0.0"
---

# GitHub Repository Setup for Hermes Playbook Backup

## Purpose
Centralized version control and backup for Hermes Agent configuration, multi-machine setup, and best practices documentation.

## When to Use
- Setting up new Hermes Agent installations
- Creating disaster recovery backups
- Syncing configurations between multiple machines (NUC ↔ Win11)
- Documenting configuration changes over time

## Key Learnings (April 2026)

### Repository Structure
- **01-setup/**: Installation guides and pre-flight checks
- **02-machines/**: Hardware-specific configurations (NUC as gateway, Win11 as workstation)
- **03-models/**: Model stack definitions (updated April 2026: Gemma 4 + Qwen3.5, removed Claude)
- **06-tools/**: Tool-specific documentation (backup, credentials, sentinel mode)
- **README.md**: Master overview with current stack status
- **.gitignore**: Excludes sensitive files (env vars, model weights, SSH keys)

### Workflow Pattern
1. Create GitHub repository with README initialization
2. Link local repo to remote: `git remote add origin <url>`
3. Commit with descriptive messages including date and changes
4. Push to GitHub: `git push -u origin master`
5. Sync between machines: `git pull` then copy files to playbook directory

### Syncthing Integration
- Config, sessions, memories sync automatically via Syncthing
- API keys (.env) intentionally NOT synced for security
- Model weights stored locally, not in git

### Model Stack (Current - April 2026)
- **Primary**: Ollama `gemma4:26b-a4b-it-q5_K_M` (local, $0 cost)
- **Fallback**: Ollama `qwen3.5:27b-it-q5_K_M` (local, $0 cost)
- Removed: Claude Sonnet 4 (Anthropic banned agent credits)

### Multi-Layer Resilience
- Layer 1: Credential pools for key rotation (401/429 handling)
- Layer 2: Primary model fallback (session-level handoff)
- Layer 3: Auxiliary routing (per-task provider configuration)

### Backup Strategy
- **Quick snapshots**: `hermes backup --quick --label "pre-update"` (~36MB, state-focused)
- **Full backups**: `hermes backup -o backup.zip` (everything, for migration)
- **Frequency**: Quick before any config change; monthly full backups

## File Locations
- Playbook: `~/HQ/HERMESHQ/hermes-playbook/`
- Backup repo: `~/Documents/hermes-playbook-backup/`
- GitHub: `github.com/YOUR_USERNAME/hermes-playbook-backup`

## Common Commands
```bash
# Create backup
hermes backup --quick --label "pre-change"

# List snapshots
ls ~/.hermes/state-snapshots/

# Restore via gateway
/snapshot restore <snapshot-id>

# Git operations
cd ~/Documents/hermes-playbook-backup
git pull origin master    # Get updates
git push origin master    # Share changes
cp -r * ~/.hermes/hermes-playbook/  # Deploy
```