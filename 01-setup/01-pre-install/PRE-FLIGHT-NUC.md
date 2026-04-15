# Pre-Flight Check: NUC (Headless Relay)

**Purpose:** Verify NUC current state and confirm it's ready for role as pure relay
**Run on:** NUC via SSH

---

## Current State Audit

### Hermes Version
```bash
hermes --version

# Expected: hermes-agent v0.8.0
# If behind: hermes update
```

### Hermes Gateway Status
```bash
hermes gateway status

# Should show: running, Telegram connected, Discord connected
```

### SSH Server
```bash
# Check SSH is running
systemctl --user status sshd

# If not running:
# sudo systemctl enable --now sshd
```

### Syncthing Status
```bash
# Check Syncthing
systemctl --user status syncthing.service

# Or check via CLI if installed
syncthing-cli status

# Verify HERMESHQ folder syncing
syncthing-cli folder <folder-id>
```

### Check What Else is Running
```bash
# List running user services
systemctl --user list-units --type=service --state=running

# Check for resource-heavy processes
ps aux --sort=-%mem | head -10
```

---

## Current Resource Usage
```bash
# Memory usage
free -h

# Disk usage
df -h

# Expected: 8GB RAM, 120GB SSD
# Memory should be <4GB in use (it's a relay, not running inference)
```

---

## SSH Key Setup (For Win11 Connection)

### Check if SSH keys exist
```bash
ls -la ~/.ssh/

# Expected: id_ed25519, id_ed25519.pub (or id_rsa variants)
```

### Check authorized_keys for Win11
```bash
cat ~/.ssh/authorized_keys

# Should contain Win11 PC public key if already set up
```

---

## Services Running (Should Only Be These)
```bash
systemctl --user list-units --type=service --state=running
```

### Expected Services
| Service | Purpose |
|---------|---------|
| hermes-gateway.service | Telegram + Discord bot |
| syncthing.service | File sync to Win11 |
| sshd | Remote access |

### Extra Services to Remove (If Present)
- Any GUI applications (foot, sway, waybar)
- Audio services (pipewire, pulseaudio)
- Browser processes

---

## Network Configuration
```bash
# Check IP address
ip addr show | grep "inet "

# Should be: 192.168.172.238 (or configured static IP)
```

### Port Check
```bash
# Check SSH on standard port
ss -tlnp | grep 22

# Check if firewall blocking
sudo iptables -L -n | grep 22
```

---

## Backup Verification

### Verify Backup Location
```bash
# Check backup exists
ls ~/HQ/HERMESHQ/BACKUPS/

# Latest backup should be: nuc-critical-backup-20260413_085226
```

### Verify Critical Files
```bash
# These should exist and be recent
ls -la ~/.hermes/config.yaml
ls -la ~/.hermes/.env
ls -la ~/.hermes/skills/
ls -la ~/HQ/HERMESHQ/BACKUPS/nuc-critical-backup-20260413_085226/
```

---

## Pre-Flight Results Table

| Check | Command | Expected | Status |
|-------|---------|----------|--------|
| Hermes Version | `hermes --version` | v0.8.0 | ⬜ |
| Gateway Status | `hermes gateway status` | Running | ⬜ |
| SSH Server | `systemctl status sshd` | Active | ⬜ |
| Syncthing | `syncthing-cli status` | Syncing | ⬜ |
| Memory Usage | `free -h` | <4GB used | ⬜ |
| SSH Keys | `ls ~/.ssh/` | Keys exist | ⬜ |
| Backup | `ls BACKUPS/` | Backup exists | ⬜ |
| Win11 SSH Key | `cat authorized_keys` | Win11 key present | ⬜ |

---

## If Any Check Fails

### Hermes behind on updates
```bash
hermes update
systemctl --user restart hermes-gateway.service
```

### Syncthing not running
```bash
systemctl --user enable --now syncthing.service
```

### Extra services running
```bash
# List all user services
systemctl --user list-units --type=service

# Stop unnecessary services
systemctl --user stop <service-name>
systemctl --user disable <service-name>
```

### Win11 SSH key not added
```bash
# On Win11, copy public key:
# type $env:USERPROFILE\.ssh\id_ed25519.pub

# On NUC, add to authorized_keys:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "PASTE_WIN11_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

---

## Next Step

When all checks pass, proceed to: `../03-nuc-post-install/01-current-state-audit.md`
