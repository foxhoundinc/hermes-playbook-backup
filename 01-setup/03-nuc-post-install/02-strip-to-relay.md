# Strip NUC to Pure Relay

**Purpose:** Remove everything from NUC that isn't relay-related
**Based on:** `01-current-state-audit.md` findings
**Run on:** NUC via SSH

---

## What to Remove

Based on the audit, remove anything that isn't:
- SSH (for NUC access)
- Hermes gateway (Telegram + Discord relay)
- Syncthing (HQ ↔ Win11 sync)
- Cron scheduler
- Systemd (service management)

### Common Things to Remove on Arch Linux NUC

```bash
# Check for Docker (heavy, not needed on NUC)
systemctl --user status docker
sudo pacman -Rns docker docker-compose 2>/dev/null || true

# Check for Node.js / npm (not needed for relay-only)
which node
sudo pacman -Rns nodejs npm 2>/dev/null || true

# Check for Python dev packages (not needed)
which python
# Keep python — Hermes needs it
# But remove unused python packages:
pip cache purge

# Check for email servers, web servers, databases
sudo pacman -Rns postfix sendmail apache2 nginx mysql 2>/dev/null || true

# Check for desktop environments (if headless, remove GUI)
# This depends on your install — be careful
# systemctl get-default
# If graphical.target → sudo systemctl set-default multi-user.target
```

---

## List of Safe Removals

| Package | Why Remove | How |
|---------|-----------|-----|
| docker | NUC can't run containers efficiently | `sudo pacman -Rns docker` |
| postfix/sendmail | Mail server not needed | `sudo pacman -Rns postfix` |
| apache2/nginx | Web server not needed | `sudo pacman -Rns nginx` |
| mysql | Database not on NUC | `sudo pacman -Rns mariadb` |
| unused fonts | Saves disk/memory | `sudo pacman -Rns $(pacman -Qqdt)` |
| unused languages | LibreOffice, etc. | Check with `pacman -Qdtt` |

**DO NOT REMOVE:** openssh, python, git, syncthing, systemd

---

## Stop Non-Essential Services

```bash
# List all running services
systemctl --user list-units --type=service --state=running

# Stop and disable anything not in this list:
ESSENTIAL_SERVICES=(
    "sshd"
    "syncthing"
    "cronie"          # or cronie for cron
    "hermes-gateway"  # if installed as service
)

# For each non-essential service:
systemctl --user stop <service-name>
systemctl --user disable <service-name>
systemctl --user mask <service-name>  # prevent accidental start
```

---

## Clean Up User Crons

```bash
# View current crontab
crontab -l

# Remove any crons that aren't:
# - Hermes cron jobs
# - Syncthing sync scripts
# - Whisper transcription scripts

crontab -e  # edit and remove
```

---

## Free Up RAM

```bash
# Clear package cache
sudo pacman -Scc

# Clear thumbnail cache
rm -rf ~/.cache/thumbnails/*

# Clear browser cache (if any)
rm -rf ~/.cache/mozilla ~/.cache/chromium

# Clear old logs
sudo journalctl --vacuum-time=7d

# Check memory after cleanup
free -h
```

---

## Set to Headless/Multi-User Mode

If NUC boots to GUI (desktop environment):

```bash
# Check current boot target
systemctl get-default

# Set to headless (no GUI)
sudo systemctl set-default multi-user.target

# Now NUC boots to CLI only — saves ~500MB RAM
```

---

## Verify

After stripping:

```bash
# Check RAM usage
free -h
# Expected: under 500MB used (from ~1.5GB with desktop)

# Check what's listening on network
ss -tlnp
# Should only show: SSH (22), Syncthing (8384), Hermes gateway ports

# Check startup services
systemctl list-unit-files --state=enabled
# Should only show: sshd, syncthing, cronie, getty@
```

---

## Result

After stripping, your NUC should:
- Use under 500MB RAM (from ~1.5GB+ with desktop)
- Boot in under 30 seconds
- Only expose SSH + Syncthing + Hermes gateway ports
- Run 24/7 with minimal power draw

---

## Next Step

After stripping:
- **03-ssh-hardening.md** — Secure SSH access
- **04-systemd-services.md** — Ensure essential services auto-start
