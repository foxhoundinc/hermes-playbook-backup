# NUC Current State Audit

**Purpose:** Document what's currently running on the NUC before making changes
**Run on:** NUC via SSH
**Critical:** Do this before stripping anything

---

## What to Check

Run these commands on the NUC and record the output.

### 1 — System Resources
```bash
# RAM and CPU
free -h
nproc
cat /proc/cpuinfo | grep "model name" | head -1

# Disk space
df -h

# Uptime
uptime
```

### 2 — OS Version
```bash
cat /etc/os-release
uname -r
```

### 3 — Hermes Installation
```bash
hermes --version
hermes gateway status
systemctl --user status hermes-gateway.service
```

### 4 — Running Services
```bash
# User services
systemctl --user list-units --type=service --state=running

# All listening ports (what's exposed)
ss -tlnp

# Active cron jobs
crontab -l
systemctl --user list-timers
```

### 5 — Syncthing
```bash
# Syncthing service
systemctl --user status syncthing.service

# Syncthing version
syncthing --version

# Syncthing config location
cat ~/.config/syncthing/config.xml | grep address
```

### 6 — SSH
```bash
# SSH service
systemctl status sshd

# SSH config
cat /etc/ssh/sshd_config | grep -E "Port|PermitRoot|PasswordAuth"

# Active SSH sessions
ss -tnp | grep :22
```

### 7 — Network
```bash
# IP addresses
ip addr

# Hostname
hostname
hostname -I
```

### 8 — Docker (if any)
```bash
docker ps
systemctl status docker
```

### 9 — Installed Packages
```bash
# Package manager
pacman -Q | wc -l

# Or if Ubuntu/Debian:
# dpkg -l | wc -l
```

### 10 — Firewalld
```bash
sudo firewall-cmd --list-all
# Or
sudo iptables -L -n
```

---

## Output Template

Copy this and fill in results:

```
=== NUC STATE AUDIT ===

Date: YYYY-MM-DD

HARDWARE:
- Model: NUC i3
- RAM: X GB
- CPU: X cores @ X.X GHz
- Disk: X GB free / X GB total
- Uptime: X days

OS:
- Distribution: Arch Linux (or Xubuntu X.X)
- Kernel: X.X.X

HERMES:
- Version: vX.XX
- Gateway: running / stopped
- Auto-start: enabled / disabled

SERVICES RUNNING:
1. hermes-gateway: running / stopped
2. syncthing: running / stopped
3. sshd: running / stopped
4. Other: ...

SYSTHING:
- Config: ~/.config/syncthing/
- Devices: [list device names/IDs]
- Folders: [list folder labels/paths]

CRON JOBS:
1. [job name] — schedule — last run: X
2. ...

NETWORK:
- Hostname: hermes-nuc
- LAN IP: 192.168.X.X
- SSH port: 22

SECURITY:
- Root login: allowed / denied
- Password auth: on / off
- Firewall: active / inactive

THINGS TO REMOVE:
1. [package/service]
2. ...

THINGS TO KEEP:
1. hermes-gateway
2. syncthing
3. sshd
4. cron
```

---

## Next Step

After completing this audit, move to:
- **02-strip-to-relay.md** — Remove everything not needed
