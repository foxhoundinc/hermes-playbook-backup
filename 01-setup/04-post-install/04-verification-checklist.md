# Full Verification Checklist

**Purpose:** Complete smoke test of both machines before declaring the setup done
**Run on:** NUC and Win11

---

## How to Use This Checklist

Run each section. Check each item. Document results. Fix any failures before declaring victory.

---

## SECTION 1 — NUC Verification

### 1.1 Basic Connectivity
```bash
# NUC reachable on LAN
ping -c 3 192.168.172.238

# NUC can reach internet
curl -s --max-time 5 https://hermesatlas.com > /dev/null && echo "OK" || echo "FAIL"
```

### 1.2 SSH Access
```bash
# From Win11:
ssh -i ~/.ssh/id_ed25519 nac_username@192.168.172.238 "hostname && uptime"
# Must succeed without password prompt
```

### 1.3 Services
```bash
# All critical services running
for svc in sshd syncthing cronie; do
    systemctl is-active $svc && echo "$svc: OK" || echo "$svc: FAIL"
done

# Hermes gateway
systemctl --user is-active hermes-gateway.service && echo "hermes-gateway: OK" || echo "hermes-gateway: FAIL"
```

### 1.4 Hermes Gateway
```bash
hermes gateway status
# Should show Telegram/Discord connected (or configured)
```

### 1.5 Syncthing
```bash
syncthing-cli status
# Should show HERMESHQ: Up to Date
```

### 1.6 Resources
```bash
free -h | grep Mem
df -h /
uptime
# Expected: <500MB RAM, <50% disk used, uptime > 0
```

---

## SECTION 2 — Win11 Verification

### 2.1 Basic Connectivity
```powershell
# Win11 reachable from NUC
ssh -i ~/.ssh/id_ed25519 win11_username@192.168.x.x "hostname"

# Win11 can reach internet
curl.exe -s --max-time 5 https://hermesatlas.com > $null; if ($LASTEXITCODE -eq 0) { "OK" } else { "FAIL" }
```

### 2.2 Hermes Agent
```powershell
hermes --version
# Expected: v0.90

hermes config validate
# Should show: Config valid
```

### 2.3 LM Studio
```powershell
# Is LM Studio installed and server running?
curl.exe http://localhost:1234/v1/models
# Should return JSON with model list

# Load a model
lms chat qwen/qwen3.5-9b "test"
# Should respond
```

### 2.4 GPU Acceleration
```powershell
nvidia-smi --query-gpu=name,memory.total,utilization.gpu --format=csv,noheader
# Should show RTX 5070 Ti, ~16GB, utilization during load
```

### 2.5 ACP Listener
```powershell
# ACP running and listening
hermes acp ping 192.168.x.x:8080
# Should show: win11-heavy online
```

### 2.6 Resources
```powershell
Get-CimInstance Win32_OperatingSystem | Select FreePhysicalMemory, TotalVisibleMemorySize
# Should show ~60GB free of 64GB

nvidia-smi --query-gpu=memory.used,memory.total --format=csv,noheader
# Should show VRAM usage
```

---

## SECTION 3 — Cross-Machine Communication

### 3.1 NUC → Win11 SSH
```bash
# From NUC:
ssh -i ~/.ssh/id_ed25519 win11_username@192.168.x.x "powershell -c 'hostname'"
# Must succeed without password
```

### 3.2 NUC → Win11 ACP
```bash
hermes acp ping 192.168.x.x:8080
# Should show: win11-heavy online
```

### 3.3 Win11 → NUC SSH (for admin access)
```powershell
ssh -i ~/.ssh/id_ed25519 nac_username@192.168.172.238 "uptime"
# Must succeed
```

### 3.4 File Transfer
```bash
# NUC → Win11
echo "test" | ssh -i ~/.ssh/id_ed25519 win11_username@192.168.x.x "cat > C:\Users\win11_username\test.txt"

# Verify
ssh -i ~/.ssh/id_ed25519 win11_username@192.168.x.x "type C:\Users\win11_username\test.txt"
# Should show: test
```

---

## SECTION 4 — Syncthing

### 4.1 NUC ← → Win11
```bash
# On NUC:
syncthing-cli devices
# Should show Win11 as connected

syncthing-cli folder herameshQ
# Should show: Up to Date
```

### 4.2 HERMESHQ Sync Test
```bash
# Create test file on NUC
echo "cross-machine test $(date)" > ~/HQ/HERMESHQ/cross-machine-test.txt
sleep 15
# Check on Win11
ssh -i ~/.ssh/id_ed25519 win11_username@192.168.x.x "type C:\HermesHome\cross-machine-test.txt"
# Should show the test content

# Clean up
rm ~/HQ/HERMESHQ/cross-machine-test.txt
```

---

## SECTION 5 — Data Integrity

### 5.1 Hermes Config on Win11
```powershell
# Verify all critical files on Win11
Test-Path C:\HermesHome\.hermes\config.yaml
Test-Path C:\HermesHome\.hermes\skills
Test-Path C:\HermesHome\.hermes\sessions
# All should return: True
```

### 5.2 API Keys
```powershell
# Verify keys are present in .env or config
Select-String -Path C:\HermesHome\.hermes\.env -Pattern "OPENROUTER_API_KEY"
# Should show key line
```

---

## SECTION 6 — Integration Tests

### 6.1 Hermes on Win11 Responds
```powershell
hermes ask "What is 2+2?"
# Should respond with 4
```

### 6.2 Local LLM via LM Studio
```powershell
hermes ask --model lmstudio:qwen/qwen3.5-9b "What is 2+2?"
# Should respond (if LM Studio server running)
```

### 6.3 Cron Jobs Listed
```powershell
hermes cron list
# Should show SideHustle, AILearn, Scanner
```

### 6.4 Whisper on NUC
```bash
# On NUC:
which whisper
# Should return path to whisper
```

---

## SECTION 7 — Stress Test

### 7.1 Local LLM Load Test
```powershell
# On Win11 — load largest model and run a task
# This verifies GPU + RAM offload works correctly
lms chat qwen/qwen3.5-27b "Summarize the principles of cognitive optimization"
# Should complete without OOM error
```

### 7.2 Concurrent ACP Tasks
```bash
# From NUC — send 2 tasks to Win11 simultaneously
hermes acp dispatch --to win11-heavy --task "echo task1" &
hermes acp dispatch --to win11-heavy --task "echo task2" &
wait
# Both should complete
```

---

## COMPLETION SIGN-OFF

When ALL sections pass:

```
===============================================
FULL VERIFICATION — ALL SECTIONS COMPLETE
===============================================

SECTION 1 — NUC: ________________ / 6
SECTION 2 — WIN11: _______________ / 6
SECTION 3 — CROSS-MACHINE: _______ / 4
SECTION 4 — SYNCTHING: ___________ / 2
SECTION 5 — DATA INTEGRITY: _______ / 2
SECTION 6 — INTEGRATION: __________ / 4
SECTION 7 — STRESS TEST: __________ / 2

TOTAL: _______ / 26

Signature: _________________  Date: ___________
Notes: _________________________________________
```

---

## If Something Fails

Fix the specific section first, then re-run that section's checks. Don't proceed to the next major section until the current one is clean.

---

## Next Step

After full verification:
- **05-first-test-run.md** — Send real prompts through the system
