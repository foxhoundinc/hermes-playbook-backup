# SSH from NUC to Win11

**Purpose:** Configure passwordless SSH from NUC to Win11 so the NUC can delegate tasks
**Architecture:** NUC (control plane) → SSH → Win11 (worker)
**Run on:** NUC (Arch Linux) and Windows 11

---

## Overview

```
NUC (hermes-nuc)                    Win11 (hermes-win11)
192.168.172.238                    192.168.x.x (LAN)
24/7 running                        Sleeps/wakes as needed
Fires cron jobs  ────SSH─────────►  Executes heavy tasks
                                         ↕
                                    LM Studio (local LLMs)
```

---

## On Win11 — Enable OpenSSH Server

### Step 1 — Install OpenSSH Server

```powershell
# Open PowerShell as Administrator
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Or via Settings:
# Settings → System → Optional Features → Add a feature → OpenSSH Server
```

### Step 2 — Start SSH Server Service

```powershell
# Start the service
Start-Service sshd

# Set to auto-start
Set-Service -Name sshd -StartupType Automatic

# Verify it's running
Get-Service sshd
```

### Step 3 — Firewall Rules

```powershell
# Allow SSH through firewall (should be automatic with install)
New-NetFirewallRule -DisplayName "SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow

# Verify
Get-NetFirewallRule | Where-Object {$_.DisplayName -eq "SSH"}
```

---

## On Win11 — Set Static IP (Recommended)

```powershell
# Find your network adapter name
Get-NetAdapter

# Set static IP (example — adjust to your network)
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.100 `
    -PrefixLength 24 -DefaultGateway 192.168.1.1

# Set DNS
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.1
```

**Note your Win11 IP — you'll need it for NUC SSH config.**

---

## On NUC — Generate SSH Key (if not already done)

```bash
# Check if you already have a key
ls ~/.ssh/

# If not, generate one
ssh-keygen -t ed25519 -C "hermes-nuc"

# Press Enter for default location
# Leave passphrase blank (or set one and use ssh-agent)
```

---

## On Win11 — Authorize NUC's SSH Key

From NUC, copy the public key:

```bash
# On NUC — copy the public key
cat ~/.ssh/id_ed25519.pub
# Output looks like: ssh-ed25519 AAAAC3Nza... hermes-nuc
```

On Win11, paste this into the authorized keys file:

```powershell
# Create .ssh directory
mkdir C:\Users\YourUsername\.ssh

# Create authorized_keys
notepad C:\Users\YourUsername\.ssh\authorized_keys

# Paste the NUC's public key (one line, no quotes)
ssh-ed25519 AAAAC3Nza... hermes-nuc
```

Permissions matter on Windows:
```powershell
# Set correct permissions
icacls C:\Users\YourUsername\.ssh\authorized_keys /inheritance:r /grant "YourUsername:F"
```

---

## On NUC — SSH Config for Easy Access

```bash
# On NUC — add to ~/.ssh/config
nano ~/.ssh/config
```

```ssh-config
# Win11 heavy worker
Host win11
    HostName 192.168.x.x        # Win11 static IP
    User YourUsername
    IdentityFile ~/.ssh/id_ed25519
    StrictHostKeyChecking accept-new
    ServerAliveInterval 60
```

Test from NUC:
```bash
# Should connect without password
ssh win11 "hostname"
```

---

## On NUC — SCP for File Transfer

```bash
# Copy files to Win11
scp ~/test.txt win11:C:\Users\YourUsername\test.txt

# Copy files from Win11
scp win11:C:\Users\YourUsername\result.txt ~/result.txt

# Sync entire directory
rsync -avz --delete ~/local/data/ win11:C:\data\
```

---

## ACP Over SSH (NUC → Win11 Task Delegation)

Once SSH works, use Hermes ACP to delegate:

```bash
# From NUC — send task to Win11
hermes acp dispatch --to win11-heavy --task "transcribe /data/podcast.mp3"

# Check status
hermes acp status

# View results
hermes acp result --task-id <id>
```

---

## Troubleshooting SSH

| Issue | Fix |
|-------|-----|
| Connection refused | Check Win11 SSH service is running |
| Permission denied (publickey) | Verify authorized_keys has NUC's public key |
| Win11 IP changes | Set static IP on Win11 router |
| NUC can't reach Win11 | Check both are on same network/subnet |
| SSH hangs on connection | Add `ServerAliveInterval 60` to SSH config |

---

## Security

- **Key-based auth only** — no passwords over SSH
- **Local LAN only** — no port forwarding or exposure to internet
- **Firewall** — only port 22 (SSH) and 8080 (ACP) open on LAN
- **NUC is the only authorized client** — only NUC's key is in authorized_keys

---

## Verify

From NUC:
```bash
# Quick connectivity test
ssh win11 "echo Win11 connection OK"

# Test file transfer
ssh win11 "hostname && whoami"
```

Expected: Returns hostname and username without asking for password.

---

## Next Step

After SSH is working:
- **07-local-llm-guide.md** — Using local models day-to-day
- **../04-post-install/03-cron-relocation.md** — Route crons to Win11 via SSH
