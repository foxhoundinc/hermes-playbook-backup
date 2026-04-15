# ACP Listener Setup — Win11 as Remote Worker

**Purpose:** Configure Win11 to accept ACP commands from NUC so the NUC can delegate heavy work
**Architecture:** NUC (24/7 relay) → SSH/ACP → Win11 (heavy execution)
**Run on:** Windows 11 PC

---

## What is ACP?

ACP (Agent Communication Protocol) lets Hermes agents on different machines talk to each other. The NUC fires a cron job, then SSH's to Win11 to execute it via ACP.

---

## Step 1 — Install ACP on Win11

```powershell
# ACP is built into Hermes Agent v0.8.0
# Verify it's available:
hermes acp --version

# If not found, update Hermes:
hermes update
```

---

## Step 2 — Generate SSH Key on Win11 (for keyless access from NUC)

```powershell
# Generate SSH key (if you haven't already)
ssh-keygen -t ed25519 -C "hermes-win11"

# Press Enter for default location
# Enter a passphrase (or leave blank)

# View your public key:
cat ~/.ssh/id_ed25519.pub
```

Copy this public key — you'll paste it into the NUC in Step 3.

---

## Step 3 — Authorize NUC's SSH Key on Win11

On Win11, add NUC's public key to authorized keys:

```powershell
# On Win11 — create SSH config
notepad C:\Users\YourUsername\.ssh\authorized_keys

# Paste the NUC's public key (from the NUC: cat ~/.ssh/id_ed25519.pub)
# Format: ssh-ed25519 AAAA... comment
```

If `authorized_keys` doesn't exist, create it.

---

## Step 4 — Configure ACP on Win11

In `D:\HermesHome\.hermes\config.yaml` on Win11:

```yaml
acp:
  enabled: true
  host: 0.0.0.0        # Listen on all interfaces
  port: 8080            # Default ACP port
  auth: key             # Key-based auth only (no password)
  workers:
    - name: win11-heavy
      platform: windows
      capabilities:
        - local-llm
        - gpu-inference
        - heavy-cron
      max_concurrent: 2

system:
  hostname: hermes-win11
```

---

## Step 5 — Open Firewall on Win11

```powershell
# Allow SSH (port 22) and ACP (port 8080) through firewall
New-NetFirewallRule -DisplayName "Hermes ACP" -Direction Inbound -Protocol TCP -LocalPort 8080 -Action Allow
New-NetFirewallRule -DisplayName "Hermes SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow
```

---

## Step 6 — Test ACP Connection from NUC

From the NUC:

```bash
# Test SSH connectivity (passwordless)
ssh -i ~/.ssh/id_ed25519 win11_username@192.168.x.x "hostname"

# If this works, test ACP:
hermes acp ping 192.168.x.x:8080
```

---

## Step 7 — Configure NUC to Use Win11 as Worker

On the NUC, edit `~/.hermes/config.yaml`:

```yaml
acp:
  workers:
    - name: win11-heavy
      host: 192.168.x.x    # Win11 IP on your LAN
      port: 8080
      auth: key
      platform: windows
      capabilities:
        - local-llm
        - gpu-inference
```

---

## Step 8 — Test Delegation

From the NUC, test sending a job to Win11:

```bash
# Send a test task to Win11 via ACP
hermes acp dispatch --to win11-heavy --task "echo test"

# Expected: Task runs on Win11, result returned to NUC
```

---

## ACP Worker Config Reference

| Parameter | Value | Notes |
|-----------|-------|-------|
| `host` | Win11 LAN IP | e.g., 192.168.1.100 |
| `port` | 8080 | Default ACP port |
| `auth` | key | SSH key auth only |
| `platform` | windows | Required |
| `capabilities` | local-llm, gpu-inference | What Win11 can do |

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Connection refused on port 8080 | Check Win11 firewall rules |
| SSH key auth fails | Verify authorized_keys on Win11 |
| ACP not recognized | Update Hermes on Win11: `hermes update` |
| Win11 IP changes | Set static IP in router or use hostname |

---

## Security Note

- ACP worker only listens on your **local LAN**
- SSH key auth only — no passwords
- No ports exposed to the internet
- NUC is the only machine authorized to send commands to Win11

---

## Next Step

After ACP is working:
- **06-ssh-from-nuc.md** — Fine-tune SSH config from NUC to Win11
- **05-config-priorities.md** — Update config.yaml with full model stack
