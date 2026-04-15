# Backup & Restore Guide

**Last Updated:** 2026-04-15  
**Current Commit:** b247e0a  

---

## 📦 Quick Backups (Automated)

Created with `hermes backup --quick`:

| Backup ID | Date | Contents |
|-----------|------|----------|
| 20260415-121618 | Today | Current state (latest) |
| 20260415-093613 | Today | Previous state |
| 20260414-085134 | Yesterday | Test snapshot |
| 20260414-024830 | Yesterday | v0.90 post-update |

### 🔄 Restore from Quick Backup

```bash
# List available snapshots
ls ~/.hermes/state-snapshots/

# Restore specific snapshot
hermes snapshot restore 20260415-121618
```

---

## 📚 Git History (Version Control)

### Current Branch: master

```
* b247e0a  Adapt profiles: Remove WhatsApp, configure for Telegram/Discord/Gemma
* 0369e4e  feat: add case sensitivity resolution skill
*   f537326  Merge GBrain: accept README.md from upstream
|\  
| * b7e3005  fix: sync pipeline, extract, features, autopilot (v0.10.1)
* | d4444d0  Initial commit - April 2026
 / 
```

### 🔄 Restore from Git

```bash
# View all commits
git log --oneline --all --graph

# Checkout current state
git checkout master

# Or restore to specific commit
git checkout b247e0a
```

---

## 💾 Full System Backup

### What Gets Backed Up

```
~/.hermes/
├── state-snapshots/    ← Quick snapshots (created by hermes backup)
├── config.yaml         ← Configuration
├── config.yaml.bak     ← Backup of config
└── state/              ← Persistent state
    └── db.sqlite       ← Main database
```

### 📝 Manual Backup Steps

```bash
# 1. Quick snapshot (automated)
hermes backup --quick

# 2. Copy config
cp ~/.hermes/config.yaml ~/.hermes/config.yaml.bak

# 3. Copy database
cp ~/.hermes/state/db.sqlite ~/.hermes/state/db.sqlite.bak

# 4. Create archive
tar -czf hermes-backup-$(date +%Y%m%d).tar.gz \
    ~/.hermes/config.yaml \
    ~/.hermes/state/ \
    ~/.hermes/state-snapshots/
```

---

## 🔧 Git Operations

### Current Repository State

```bash
# Status (should be clean)
git status
# On branch master, nothing to commit, working tree clean

# Remote repositories
git remote -v
# origin  https://github.com/foxhoundinc/hermes-playbook-backup.git
# gbrain  https://github.com/garrytan/gbrain

# View commit history
git log --oneline -5
```

### Restoring from Git

```bash
# Pull latest changes
git pull origin master

# Or checkout specific commit
git checkout b247e0a

# Restore after problems
git checkout -- .
```

---

## 🎯 Recommended Backup Strategy

### For Production Use:

1. **Daily:** Automated quick backups (`hermes backup --quick`)
2. **Weekly:** Full git backup (`git push origin master`)
3. **Monthly:** Archive snapshot + config + database

### Before Major Changes:

```bash
# 1. Create quick backup
hermes backup --quick

# 2. Commit to git
git add .
git commit -m "Pre-change backup - 2026-04-15"

# 3. Push to remote
git push origin master
```

### Recovery Scenarios:

| Scenario | Recovery Method |
|----------|-----------------|
| Configuration mistake | `git checkout -- config.yaml` |
| State corruption | `hermes snapshot restore 20260415-121618` |
| Complete rebuild | Git checkout + restore snapshots |
| Profile issues | Restore from `~/.hermes/skills/` backup |

---

## 📋 Current Backup Inventory

### Quick Snapshots (4 total)
- ✅ 20260415-121618 (Latest - Recommended for restore)
- ✅ 20260415-093613
- ✅ 20260414-085134
- ✅ 20260414-024830

### Git Commits (4 total)
- ✅ b247e0a (Current - Profile adaptations)
- ✅ 0369e4e (Case sensitivity fix)
- ✅ f537326 (GBrain merge)
- ✅ d4444d0 (Initial commit)

### Remote Backups
- ✅ GitHub: `foxhoundinc/hermes-playbook-backup`
- ✅ Gbrain: `garrytan/gbrain` (reference)

---

## 💡 Pro Tips

1. **Always create a snapshot before major changes**
2. **Keep at least 3 recent snapshots**
3. **Commit significant changes to git immediately**
4. **Push to remote at least once per week**
5. **Test restores periodically**

### Test Restore:
```bash
# Test without affecting current state
hermes snapshot restore 20260415-121618 --dry-run
```

---

**Backup Status:** ✅ All systems backed up and ready for restore
**Last Backup:** 2026-04-15 22:16:18
**Next Backup:** Recommended before any major changes

