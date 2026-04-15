# Pre-Flight Check: Windows 11 PC

**Purpose:** Verify Win11 PC hardware and software is ready before Hermes installation
**Run on:** Windows 11 PC (PowerShell as Administrator)

---

## Hardware Verification

Open PowerShell and run each command. All should return values/versions.

### GPU & CUDA
```powershell
# Check GPU recognized
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv,noheader

# Expected output: RTX 5070 Ti, driver version, ~16GB memory
```

### RAM
```powershell
# Total physical memory
systeminfo | findstr /C:"Total Physical Memory"

# Expected: 64GB+
```

### Disk Space
```powershell
# Check C: drive free space
Get-PSDrive C | Select-Object Name, @{Name="FreeGB";Expression={"{0:N2}" -f ($_.Free/1GB)}}

# Expected: 100GB+ free for Hermes + LM Studio + models
```

---

## Software Prerequisites

### Python
```powershell
python --version

# Expected: Python 3.11 or higher
# If not installed: winget install Python.Python.3.11
```

### Git
```powershell
git --version

# Expected: git version 2.x.x
# If not installed: winget install Git.Git
```

### Node.js
```powershell
node --version
npm --version

# Expected: v20.x.x or higher
# If not installed: winget install OpenJS.NodeJS
```

### CUDA Toolkit
```powershell
nvcc --version

# Expected: CUDA 12.x
# If not installed: Download from https://developer.nvidia.com/cuda-downloads
# IMPORTANT: Reboot after CUDA install
```

---

## Windows Defender Exclusions (CRITICAL)

Run PowerShell as Administrator:

```powershell
# Add exclusions for Hermes directories
Add-MpPreference -ExclusionPath "C:\Hermes"
Add-MpPreference -ExclusionPath "C:\Users\$env:USERNAME\.hermes"

# Verify exclusions added
Get-MpPreference | Select-Object -ExpandProperty ExclusionPath
```

**Expected:** Should list both paths without errors.

---

## Visual Studio Build Tools

Required for Python packages with C/C++ extensions (pywinpty, faster-whisper, etc.)

```powershell
# Check if installed
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-VC-BuildTools

# If not installed:
# 1. Download from: https://visualstudio.microsoft.com/downloads/
# 2. Under "Tools for Visual Studio" → "Build Tools for Visual Studio 2022"
# 3. Select: "Desktop development with C++"
# 4. Install (~6GB)
```

---

## Check SSH Client
```powershell
# Check OpenSSH client
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

# If not installed:
# Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

---

## Network Check
```powershell
# Test connectivity to NUC
Test-Connection -ComputerName 192.168.172.238 -Count 2

# Expected: Reply from 192.168.172.238
```

---

## LM Studio Compatibility Check

Your RTX 5070 Ti is compatible with LM Studio. Verify:

```powershell
# Check DirectX/Vulkan support
dxdiag | Select-String -Pattern "DirectX Version"

# Expected: DirectX 12 or higher
```

---

## Pre-Flight Results Table

| Check | Command | Expected | Status |
|-------|---------|----------|--------|
| GPU | `nvidia-smi` | RTX 5070 Ti, 16GB | ⬜ |
| RAM | `systeminfo` | 64GB+ | ⬜ |
| Python | `python --version` | 3.11+ | ⬜ |
| Git | `git --version` | 2.x+ | ⬜ |
| Node.js | `node --version` | v20+ | ⬜ |
| CUDA | `nvcc --version` | 12.x | ⬜ |
| Defender Exclusions | PowerShell | No errors | ⬜ |
| VS Build Tools | GUI check | Installed | ⬜ |
| SSH Client | PowerShell | Available | ⬜ |
| NUC Connectivity | `Test-Connection` | Success | ⬜ |

---

## If Any Check Fails

### Python not installed
```powershell
winget install Python.Python.3.11
```

### Git not installed
```powershell
winget install Git.Git
```

### Node.js not installed
```powershell
winget install OpenJS.NodeJS
```

### CUDA not installed
1. Download from https://developer.nvidia.com/cuda-downloads
2. Select: Windows > 11 > x86_64 > exe(local)
3. Install, then **REBOOT**
4. Verify with `nvidia-smi`

### Defender blocking Hermes
```powershell
# MUST run as Administrator
Add-MpPreference -ExclusionPath "C:\Hermes"
Add-MpPreference -ExclusionPath "$env:USERPROFILE\.hermes"
```

---

## Next Step

When all checks pass, proceed to: `../02-win11-install/01-prerequisites.md`
