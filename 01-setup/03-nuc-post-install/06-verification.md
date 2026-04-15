# NUC Relay Verification

**Purpose:** Full smoke test that the NUC relay is working correctly
**Run on:** NUC via SSH

---

## Pre-Flight Checklist

Complete this entire checklist before declaring the NUC ready.

---

## 1 — Basic Connectivity

```bash
# NUC is reachable
ping -c 3 192.168.172.238

# From Win11:
ping -c 3 192.168.172.238
# Both should succeed
```

---

## 2 — SSH Access

```bash
# From Win11 PowerShell:
ssh -i ~/.ssh/id_ed25519 nac_username@192.168.172.238 "hostname && uptime"

# Should return NUC hostname and uptime without password prompt
```

---

## 3 — Services Running

```bash
# All critical services
systemctl --user is-active syncthing.service && echo "syncthing: OK" || echo "syncthing: FAIL"
systemctl --user is-active hermes-gateway.service && echo "hermes-gateway: OK" || echo "hermes-gateway: FAIL"
systemctl is-active cronie && echo "cronie: OK" || echo "cronie: FAIL"
systemctl is-active sshd && echo "sshd: OK" || echo "sshd: FAIL"
```

---

## 4 — Hermes Gateway

```bash
# Check gateway status
hermes gateway status

# Should show:
# Telegram: connected (or disabled if not configured)
# Discord: connected (or disabled if not configured)
# Uptime: X hours

# Test gateway via CLI
hermes ask "ping"

# Gateway should respond (even if just "pong")
```

---

## 5 — Syncthing

```bash
# Check Syncthing status
syncthing-cli devices

# Verify NUC can see Win11 as a Syncthing device
syncthing-cli status

# Check folder sync status
syncthing-cli folders

# Expected: HERMESHQ folder showing "syncing" or "up to date"
# NOT: "stopped" or errors
```

---

## 6 — ACP Connection

```bash
# NUC can ping Win11 ACP
hermes acp ping 192.168.x.x:8080

# Should return:
# Worker: win11-heavy
# Status: online
```

---

## 7 — Cron Jobs

```bash
# Check cron is running
systemctl is-active cronie

# List active crons
crontab -l

# Run a test cron manually
hermes cron run --job-id <test-job>

# Check it executed
hermes cron history | head -10
```

---

## 8 — Whisper (Audio Transcription)

```bash
# Whisper is available
which whisper

# Test transcription with a short audio clip
whisper test_audio.mp3 --model base --language en

# Should produce test_audio.txt
```

---

## 9 — Resources

```bash
# RAM usage (should be under 500MB)
free -h | grep Mem

# Disk space
df -h /

# CPU load
uptime

# Expected: ~300-400MB RAM, <10% CPU idle, 80%+ disk free
```

---

## 10 — Full Health Script

Run the health script from `04-systemd-services.md`:

```bash
~/.local/bin/nuc-health.sh
```

Expected output:
```
=== NUC Health ===
--- Services ---
syncthing: OK
hermes-gateway: OK
cronie: OK
--- Resources ---
Mem: 1.2G / 7.7G
/dev/sda1: 45% used
--- Uptime ---
22:15:01 up 3 days, 14:30, 2 users, load average: 0.10, 0.15, 0.20
```

---

## 11 — Reboot Test

```bash
# Reboot and immediately try to reconnect after 30 seconds
sudo reboot

# From Win11:
sleep 35 && ssh -i ~/.ssh/id_ed25519 nac_username@192.168.172.238 "echo NUC is back"
```

---

## Win11 Worker Verification

```bash
# From NUC — verify Win11 responds to ACP
hermes acp dispatch \
  --to win11-heavy \
  --timeout 30 \
  --task "echo test"

# Should complete without error
```

---

## Sign-Off

When all 11 checks pass, the NUC relay is verified:

```
NUC VERIFICATION COMPLETE

[ ] Basic connectivity
[ ] SSH access
[ ] Services running
[ ] Hermes gateway
[ ] Syncthing
[ ] ACP connection
[ ] Cron jobs
[ ] Whisper
[ ] Resources OK
[ ] Health script OK
[ ] Reboot test

All boxes checked = NUC relay READY
```

---

## If Something Fails

- SSH issues → `03-ssh-hardening.md` troubleshooting
- Service won't start → `journalctl --user -u <service> -n 50`
- Syncthing problems → `systemctl --user restart syncthing`
- Gateway issues → `hermes gateway restart`
- ACP issues → check Win11 firewall and ACP service

---

## Next Step

After NUC verification is complete:
- **../04-post-install/01-syncthing-hardening.md** — Verify HERMESHQ folder syncing correctly
