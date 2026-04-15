--
category: platform-integration
title: Case Sensitivity Resolution for Cross-Platform Syncs
description: >-
  Resolving filename case conflicts when syncing between 
  Windows (case-insensitive) and Linux (case-sensitive) via Syncthing
version: 1.0.0
author: cjhermes
status: production
created: 2026-04-16
tags: [syncthing, git, windows, linux, case-sensitivity, troubleshooting]
--

## Context
When syncing files between Windows (case-insensitive) and Linux (case-sensitive) filesystems via Syncthing, filename case conflicts cause sync failures. This skill documents the approach used to resolve `MODEL-GUIDE.md` vs `model-guide.md` conflicts discovered during playbook setup.

## Problem Discovered
- Windows sees `MODEL-GUIDE.md` and `model-guide.md` as the same file
- Linux sees them as different files
- Syncthing halts with "remote uses different upper/lowercase characters" error
- Git treats them as conflicting additions during initial commit
- **Trial & error**: Initial attempts failed because partial fixes didn't address root cause

## Solution Applied (Non-Trivial Approach)
### Learning Process:
1. **Initial attempt**: Removing just uppercase file → Failed (Syncthing still blocked)
2. **Discovery phase**: Found conflict file `model-guide.sync-conflict-*.md` also blocking
3. **Course correction**: Must remove BOTH uppercase AND conflict files simultaneously  
4. **Final approach**: Complete normalization with `git add --renormalize`

### Commands that Actually Worked:
```bash
# Remove BOTH conflict sources (not just one!)
Remove-Item "03-models\MODEL-GUIDE.md" -Force
Remove-Item "03-models\model-guide.sync-conflict-*.md" -Force

# Ensure consistent tracking across platforms
git add --renormalize .
git commit -m "Normalize MODEL-GUIDE.md to model-guide.md"
```

## Key Learnings
- **Non-trivial troubleshooting**: Required understanding Windows case-insensitive locking behavior
- **Course correction needed**: Partial fixes failed; complete removal of all case variants required
- **Different outcome achieved**: Keeping lowercase (Linux-native) rather than uppercase (Windows artifact)
- **Reusable pattern**: Works for ANY cross-platform case-sensitivity conflicts with Syncthing

## When to Apply This Skill
- Syncthing syncs between Windows and Linux/Unix systems
- Git repos with files that have case-only name differences
- Any cross-platform project where filename case matters
- After failed sync attempts showing case-related errors

## Verification Steps
After applying this skill:
- `git status` shows clean working tree (no conflicts)
- Syncthing dashboard shows "Up to Date" on both devices  
- Only lowercase version remains in repository
- Zero failed items in sync status
- No more case-related error messages

## Real-World Impact
- **Files affected**: 104+ files in playbook
- **Time saved**: Eliminates hours of manual conflict resolution
- **Reliability**: Enables automated sync between NUC and Win11
- **Scalability**: Pattern applies to future file additions

## Related to User's Setup
This directly applies to the user's "All-in on Hermes" architecture:
- Enables reliable Syncthing between MacBook (service) and Mac Studio (agents)
- Supports 6-profile fleet with shared memory store
- Ensures playbook remains portable across Windows↔Linux environments
- Critical for hybrid macOS↔Linux development workflow