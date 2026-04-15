# Systemd Services — NUC Auto-Start

**Purpose:** Ensure essential services start automatically on NUC boot
**Run on:** NUC via SSH

---

## Services That Must Auto-Start

| Service | Purpose | Critical? |
|---------|---------|-----------|
| sshd | Remote access | YES |
| syncthing | File sync hub | YES |
| hermes-gateway | Telegram + Discord relay | YES |
| cronie | Cron scheduler | YES |
| systemd-networkd | Network | YES |

---

## 1 — Syncthing Systemd Service

Syncthing on Arch Linux should run as a user service.

```bash
# Enable Syncthing user service (starts on login)
systemctl --user enable syncthing.service

# Start it now
systemctl --user start syncthing.service

# Check status
systemctl --user status syncthing.service
```

Make sure lingering is enabled (so service runs even without user logged in):
```bash
loginctl enable-linger $USER

# Verify
loginctl show-user $USER | grep Linger
```

---

## 2 — Hermes Gateway Systemd Service

Create a systemd user service for Hermes gateway:

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/hermes-gateway.service
```

```ini
[Unit]
Description=Hermes Gateway Service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/hermes gateway start
Restart=always
RestartSec=10
Environment="TERM=xterm-256color"

[Install]
WantedBy=default.target
```

```bash
# Enable and start
systemctl --user daemon-reload
systemctl --user enable hermes-gateway.service
systemctl --user start hermes-gateway.service

# Check it started
systemctl --user status hermes-gateway.service
```

---

## 3 — Cronie (Cron Scheduler)

```bash
# Install cronie if not already
sudo pacman -S cronie

# Enable and start
sudo systemctl enable cronie
sudo systemctl start cronie

# Check
systemctl status cronie
```

---

## 4 — Network

Arch Linux uses systemd-networkd by default. Verify:

```bash
# Check network config
networkctl status

# If using DHCP (normal):
# Should have a .network file in /etc/systemd/network/
cat /etc/systemd/network/eth0.network   # adjust interface name
```

Example DHCP config:
```ini
[Match]
Name=eth0

[Network]
DHCP=yes
```

---

## 5 — Verify All Services at Boot

```bash
# Check what will start on boot
systemctl list-unit-files --state=enabled --user

# Should show:
# - syncthing.service (user)
# - hermes-gateway.service (user)
# - cronie.service (system)
# - sshd.service (system)
```

---

## 6 — Reboot Test

After configuring all services:

```bash
# Reboot NUC
sudo reboot

# Wait 30 seconds, then SSH back in
ssh nac_username@192.168.x.x

# Check services came up
systemctl --user status syncthing.service
systemctl --user status hermes-gateway.service
systemctl status cronie

# Check Syncthing is actually syncing
syncthing-cli devices
syncthing-cli folders
```

---

## 7 — Log Management

Prevent logs from filling up the small SSD:

```bash
# Set log rotation
sudo journalctl --vacuum-time=7d

# Or set in /etc/systemd/journald.conf:
sudo nano /etc/systemd/journald.conf
```

```
[Journal]
SystemMaxUse=256M
SystemMaxFileSize=32M
```

```bash
sudo systemctl restart systemd-journald
```

---

## Service Health Check Script

Create a quick health check:

```bash
nano ~/.local/bin/nuc-health.sh
```

```bash
#!/bin/bash
echo "=== NUC Health ==="
echo ""
echo "--- Services ---"
systemctl --user is-active syncthing.service && echo "syncthing: OK" || echo "syncthing: FAIL"
systemctl --user is-active hermes-gateway.service && echo "hermes-gateway: OK" || echo "hermes-gateway: FAIL"
systemctl is-active cronie && echo "cronie: OK" || echo "cronie: FAIL"
echo ""
echo "--- Resources ---"
free -h | grep Mem
df -h / | tail -1
echo ""
echo "--- Uptime ---"
uptime
```

```bash
chmod +x ~/.local/bin/nuc-health.sh
nuc-health.sh
```

---

## If a Service Fails to Start

```bash
# View logs for a service
journalctl --user -u hermes-gateway.service -n 50

# Check for errors
journalctl --user -u hermes-gateway.service -p err

# Manual start for debugging
systemctl --user start hermes-gateway.service
# Watch output
```

---

## Next Step

After services are stable:
- **05-acp-worker-config.md** — Configure Win11 as remote worker in NUC config
- **06-verification.md** — Full relay verification checklist
