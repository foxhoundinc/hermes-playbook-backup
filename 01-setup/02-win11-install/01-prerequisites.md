# Prerequisites Check

**Purpose:** Verify your PC has everything needed before starting the Win11 setup
**Run on:** Windows 11 PC
**Time needed:** 20-30 minutes

---

## What You Need

### Hardware (your setup)
- Windows 11 PC (Ryzen 9 9950X / RTX 5070 Ti / 64GB RAM)
- 2TB C: drive (Crucial T705 NVMe) + 4TB D: drive (Samsung 990 EVO PLUS)
- Internet connection

---

## IMPORTANT — Use WSL2 from the Start

**The recommended install path for Hermes on Windows is inside WSL2 (Ubuntu), NOT native Windows.**

Why:
- Hermes is a Linux-native tool — runs cleaner in its natural environment
- Avoids Windows path translation issues with cron jobs, file paths, and environment variables
- WSL2 GPU passthrough (WDDM) gives full RTX 5070 Ti access for local model inference — no performance loss
- Cron jobs, virtualenvs, and tool paths behave like a real Linux host
- Your API costs stay the same — same LM Studio, same GPU, same inference speed

Windows-native install (PowerShell) still works as a fallback, but you'll hit path edge cases. Start with WSL2 and save yourself the troubleshooting.

---

## Step 1 - Enable WSL2

Open **PowerShell as Administrator** and run:

```powershell
# Enable WSL2 and Virtual Machine Platform
wsl --install --enable-nested-virtualization
```

WSL2 itself is Windows infrastructure — but the Linux distro you run inside it is your choice.

**We recommend CachyOS Desktop (Arch Linux)** instead of the default Ubuntu:
- Same OS family as your NUC (Arch Linux) — consistent tooling across your whole stack
- CachyOS-specific WSL2 build with kernel optimizations and clean configuration
- Uses `pacman` — same package manager as your NUC
- Available at: https://github.com/cuppycup/cachyos-wsl

```powershell
# After WSL2 is installed, install CachyOS from the AUR-like repo
wsl --install -d cachyos
# Or if the official CachyOS WSL installer is available:
# irm https://raw.githubusercontent.com/cuppycup/cachyos-wsl/main/install.ps1 | iex
```

Restart your PC when prompted.

If you prefer the default Ubuntu:
```powershell
wsl --install --enable-nested-virtualization
# Ubuntu will be installed as the default distro
```

---

## Step 2 - Verify WSL2 and Ubuntu

After restart, Ubuntu should auto-launch. If not, open Ubuntu from the Start Menu.

Check WSL2 is working:
```bash
wsl --status
# Expected: Default Distribution: Ubuntu, Version: 2.x.x
```

Check GPU passthrough is active:
```bash
# Should show your GPU — proves WDDM driver is passing through
nvidia-smi
```

If `nvidia-smi` fails inside WSL2: see **07-troubleshooting/INBOX.md** → WSL2 Ubuntu fix.

---

## Step 3 - Set Up Ubuntu Inside WSL2

```bash
# Create a working directory (replace <you> with your Windows username)
mkdir -p /mnt/c/HermesHome
export HERMES_HOME=/mnt/c/HermesHome

# Make it permanent — add to ~/.bashrc
echo 'export HERMES_HOME=/mnt/c/HermesHome' >> ~/.bashrc
echo 'export PATH="$HERMES_HOME/hermes-agent/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## Step 5 - Check Python (inside CachyOS)

```bash
python --version
# Expected: Python 3.11 or higher
```

If not installed or wrong version:
```bash
# CachyOS uses pacman
sudo pacman -S python python-pip python-venv
```

---

## Step 6 - Check Git (inside CachyOS)

```bash
git --version
# Expected: git version 2.x.x
```

If not installed:
```bash
sudo pacman -S git
```

---

## Step 7 - Check Node.js (inside CachyOS — optional, for some tools)

```bash
node --version
# Expected: v20.x or higher
```

If not installed:
```bash
sudo pacman -S nodejs npm
```

---

## Step 8 - Verify GPU Passthrough (WSL2 → Windows RTX 5070 Ti)

```bash
nvidia-smi
# Expected: "NVIDIA GeForce RTX 5070 Ti, 16384 MB, 536.xx"
```

This confirms the WDDM driver on the Windows side is being passed through to WSL2. No separate CUDA install needed inside WSL2 — it uses the Windows host's driver.

If this fails: stop here and fix WSL2 GPU passthrough first. See **07-troubleshooting/INBOX.md** → WSL2 Ubuntu fix.

---

## Step 9 - NVIDIA Driver + CUDA Toolkit (Windows Host — already done)

These live on the **Windows side only**. You do NOT need to install them inside WSL2:

- **NVIDIA Driver** → installed on Windows (GeForce Experience or manual install)
- **CUDA Toolkit** → installed on Windows (for `nvcc` compilation on the host)

WSL2 accesses both via GPU passthrough. `nvidia-smi` works in WSL2 without a separate install.

---

## Quick Checklist (inside CachyOS)

```bash
echo "=== PREREQUISITES CHECK (CachyOS) ==="
python --version || echo "Python: MISSING"
git --version || echo "Git: MISSING"
node --version || echo "Node.js: MISSING"
nvidia-smi || echo "GPU passthrough: FAILED"
echo "All present = ready to install Hermes"
```

---

## What to Install if Missing

| Missing | How to install | Where |
|---------|---------------|-------|
| WSL2 | `wsl --install` in PowerShell (Admin) | Windows |
| CachyOS | `wsl --install -d cachyos` (see above) | WSL2 |
| Python | `sudo pacman -S python python-pip python-venv` | CachyOS (WSL2) |
| Git | `sudo pacman -S git` | CachyOS (WSL2) |
| Node.js | `sudo pacman -S nodejs npm` | CachyOS (WSL2) |
| NVIDIA Driver | nvidia.com / GeForce Experience | Windows |
| CUDA Toolkit 12 | developer.nvidia.com/cuda-downloads | Windows |

VS Build Tools (MSVC) are **not needed** inside WSL2. Pip installs pre-built wheels for almost all Python packages.

---

## After This

Once all checks pass, move to:
- **02-lm-studio-setup.md** — Install LM Studio on Windows, download models