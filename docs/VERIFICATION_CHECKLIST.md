# Verification Checklist - Hermes Installation

**Purpose:** Confirm each installation step works correctly
**Target Systems:** Windows → WSL2 → CachyOS → Hermes

---

## ✅ PREREQUISITES CHECKLIST

| Step | Item | Status | Notes |
|------|------|--------|-------|
| ☐ | Windows 11/10 with WSL enabled | | Run: `wsl --list` |
| ☐ | CachyOS ISO downloaded | | https://cachyos.org |
| ☐ | Git installed | | `git --version` |
| ☐ | Ollama installed | | `ollama --version` |
| ☐ | SSH keys configured (optional) | | For GitHub access |

---

## 📋 INSTALLATION VERIFICATION STEPS

### **Step 1: WSL2 Installation**
```bash
# Verify WSL installed
wsl --list --verbose

# Expected output:
# NAME                   STATE           VERSION
# * CachyOS                Running         2
```

**Verification:** ✅ WSL shows CachyOS running

---

### **Step 2: CachyOS Network Configuration**
```bash
# Check WSL config
cat ~/.wsl.conf

# Expected content:
# [network]
# useDHCP = true
```

**Verification:** ✅ Network config exists and valid

---

### **Step 3: Hermes Repository Clone**
```bash
# Verify clone
cd ~
ls -la hermes-playbook/

# Expected directories:
# 01-setup/  02-machines/  03-models/  04-skills/
# 05-crons/  06-tools/    07-troubleshooting/
```

**Verification:** ✅ Repository cloned with all directories

---

### **Step 4: Ollama Models Installed**
```bash
# List models
ollama list

# Expected output:
# NAME                           ID              SIZE    MODIFIED
# gemma4:26b-a4b-it-q5_K_M       ...             17GB    2 days ago
# gemma4-31b-it-4bit             ...             19GB    2 days ago
```

**Verification:** ✅ Both Gemma models listed

---

### **Step 5: Configuration Files**
```bash
# Verify config exists
cat ~/.hermes/config.yaml

# Expected content:
# display.theme: default
# routing.fallback: local
# models.gemma.path: gemma4:26b-a4b-it-q5_K_M
```

**Verification:** ✅ Config file present and valid

---

### **Step 6: Hermes Initialization**
```bash
# Check version
hermes --version

# Expected: Hermes v260408 or similar
```

**Verification:** ✅ Hermes CLI responds with version

---

## 🔍 TEST COMMANDS

### **Test 1: Profile Listing**
```bash
hermes profiles list

# Expected: 6 profiles listed
# - pmax-coder
# - pmax-ops-observer
# - pmax-content
# - pmax-dareen
# - pmax-tarek
# - pmax-mousa
```

**Result:** ☐ Not tested

### **Test 2: Profile Test**
```bash
hermes profiles test pmax-coder

# Expected: Profile loads without errors
```

**Result:** ☐ Not tested

### **Test 3: Model Inference**
```bash
# Test local model
hermes exec --profile pmax-coder -- "hello world"

# Expected: Response generated locally (no API keys)
```

**Result:** ☐ Not tested

### **Test 4: Memory Sync**
```bash
# Check memory store
ls ~/.hermes/memory/

# Expected: mem0.db present
```

**Result:** ☐ Not tested

---

## 📊 INSTALLATION STATUS SUMMARY

| Component | Status | Details |
|-----------|--------|---------|
| WSL2 | ☐ Pending | Install CachyOS |
| CachyOS | ☐ Pending | Follow wsl --import |
| Hermes Repo | ✅ Complete | Cloned and ready |
| Ollama | ✅ Complete | Installed |
| Gemma Models | ✅ Complete | Downloaded |
| Config Files | ✅ Complete | All configured |
| Hermes CLI | ✅ Complete | Version confirmed |
| Telegram Bot | ☐ Pending | Setup required |
| Discord Bot | ☐ Pending | Setup required |
| Memory Store | ✅ Complete | mem0 initialized |

---

## 🚨 KNOWN ISSUES & RESOLUTIONS

| Issue | Cause | Resolution |
|-------|-------|------------|
| WSL import fails | Large image file | Use wsl --import with correct path |
| Models not found | Ollama not running | Start Ollama service |
| Profile errors | Missing config | Run hermes init |
| Network timeouts | DNS issues | Check WSL networking |

---

## 📷 SCREENSHOT INSTRUCTIONS

**For each installation step, capture:**

1. **WSL Installation:**
   - Command: `wsl --list --verbose`
   - Show: CachyOS in "Running" state
   - Save as: `step1_wsl_status.png`

2. **CachyOS Desktop:**
   - Show: Desktop after first boot
   - Save as: `step2_cachyos_desktop.png`

3. **Ollama Models:**
   - Command: `ollama list`
   - Show: Both Gemma models listed
   - Save as: `step3_ollama_models.png`

4. **Hermes Version:**
   - Command: `hermes --version`
   - Show: Hermes CLI version
   - Save as: `step4_hermes_version.png`

5. **Profile Test:**
   - Command: `hermes profiles list`
   - Show: 6 profiles listed
   - Save as: `step5_profiles_list.png`

6. **Memory Store:**
   - Command: `ls ~/.hermes/`
   - Show: mem0.db present
   - Save as: `step6_memory_store.png`

---

## 🎯 COMPLETION CHECKLIST

- [ ] WSL2 installed and running
- [ ] CachyOS installed in WSL2
- [ ] Network connectivity verified
- [ ] Hermes repository cloned
- [ ] Ollama installed
- [ ] Gemma models downloaded
- [ ] Configuration files created
- [ ] Hermes CLI initialized
- [ ] All 6 profiles functional
- [ ] Telegram bot configured
- [ ] Discord bot configured
- [ ] Memory store operational
- [ ] Test commands passing

**Final Verification:** Run all test commands and confirm 100% success rate

