# SSH Hardening — NUC

**Purpose:** Secure SSH access to the NUC relay
**Run on:** NUC via SSH (as root or sudo)

---

## 1 — Key-Based Auth Only (Critical)

Password auth is a security risk. Disable it.

```bash
# Edit SSH server config
sudo nano /etc/ssh/sshd_config
```

Set these values:

```sshd_config
# Disable password authentication
PasswordAuthentication no

# Disable root login
PermitRootLogin no

# Disable empty passwords
PermitEmptyPasswords no

# Use strong key exchange
KexAlgorithms curve25519-sha256

# Use strong ciphers
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com

# Only allow SSH protocol 2
Protocol 2

# Disable agent forwarding unless needed
AllowAgentForwarding no

# Disable TCP forwarding
DisableForwarding no

# Keep alive to prevent dropped connections
ClientAliveInterval 300
ClientAliveCountMax 2
```

Apply changes:
```bash
sudo systemctl restart sshd
```

---

## 2 — Allow Only Specific Clients

```bash
# Add to /etc/hosts.allow
sshd: 192.168.0.0/24

# Deny all else
sshd: ALL

# In /etc/hosts.deny (should already exist)
sshd: ALL
```

Or use firewall:
```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-source=192.168.0.0/24
sudo firewall-cmd --reload
```

---

## 3 — Change Default SSH Port (Optional)

```bash
# Edit /etc/ssh/sshd_config
Port 2222   # instead of 22
```

Then update firewall:
```bash
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --reload

# Update any SSH config on other machines:
# Host nac
#     HostName 192.168.x.x
#     Port 2222
```

---

## 4 — Fail2Ban (Brute Force Protection)

```bash
# Install fail2ban
sudo pacman -Syu fail2ban

# Enable and start
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Configure
sudo nano /etc/fail2ban/jail.local
```

```jail_local
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
```

```bash
sudo systemctl restart fail2ban
```

---

## 5 — Limit Active SSH Sessions

```bash
# Max sessions per user
MaxSessions 2

# In /etc/ssh/sshd_config
```

---

## 6 — Verify NUC's Own SSH Key

```bash
# On NUC — generate host key if missing
sudo ssh-keygen -A

# Check fingerprint of your known hosts entry
ssh-keyscan -H 192.168.x.x
```

---

## 7 — Add Your Admin Key to NUC (for your own access)

```bash
# From your Mac/PC — copy your key to NUC
ssh-copy-id -i ~/.ssh/id_ed25519.pub nac_username@192.168.x.x

# Or manually:
# cat ~/.ssh/id_ed25519.pub → paste into NUC ~/.ssh/authorized_keys
```

---

## 8 — Test from Win11

```powershell
# On Win11 — test SSH to NUC
ssh -i ~/.ssh/id_ed25519 -p 2222 nac_username@192.168.x.x

# Should connect without password
```

---

## 9 — Document the NUC's SSH Info

Add to your notes:

```
NUC SSH:
- IP: 192.168.x.x
- Port: 22 (or 2222 if changed)
- User: [your username]
- Key: ~/.ssh/id_ed25519.pub (on your admin machine)
- Password auth: DISABLED
```

---

## Verify Hardening

```bash
# Test that password auth is rejected
ssh -o PreferredAuthentications=password nac_username@192.168.x.x
# Expected: Permission denied (immediately, no password prompt)

# Test that key auth works
ssh -i ~/.ssh/id_ed25519 nac_username@192.168.x.x
# Expected: Connect without password

# Check fail2ban
sudo fail2ban-client status
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Locked out of SSH | Use local console access |
| Key auth not working | Check authorized_keys format (one line, correct permissions: 600) |
| fail2ban blocking you | Wait out ban time or `sudo fail2ban-client unban <ip>` |
| Port 22 blocked | Check firewall: `sudo firewall-cmd --list-all` |

---

## Next Step

After hardening:
- **04-systemd-services.md** — Ensure services auto-start on boot
