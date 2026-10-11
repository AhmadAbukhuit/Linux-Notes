# WSL 2 Complete Setup & Optimization Guide

A comprehensive, battle-tested reference for installing, configuring, and optimizing **Windows Subsystem for Linux 2 (WSL 2)** for production software engineering, DevOps, and machine learning.

---

## 📑 Table of Contents

1. [Understanding WSL 2 Architecture](#-1-understanding-wsl-2-architecture)
2. [Prerequisites & Installation](#-2-prerequisites--installation)
3. [Core Configuration Files](#️-3-core-configuration-files)
   - [Host-wide: `.wslconfig` (Windows side)](#1-host-wide-configuration-wslconfig)
   - [Distro-specific: `/etc/wsl.conf` (Linux side)](#2-distro-level-configuration-etcwslconf)
4. [Filesystem Performance: The Golden Rule](#-4-filesystem-performance-the-golden-rule)
5. [Development Tooling Integration](#️-5-development-tooling-integration)
   - [VS Code Remote - WSL](#vs-code-remote---wsl)
   - [Git & SSH Configuration](#git--ssh-credential-sharing)
   - [Windows Terminal & Nerd Fonts](#windows-terminal--nerd-fonts)
6. [Docker in WSL 2: Two Approaches](#-6-docker-in-wsl-2-two-approaches)
7. [GPU Acceleration (NVIDIA CUDA Setup)](#-7-gpu-acceleration-nvidia-cuda-setup)
8. [Maintenance, Backups & Disk Management](#-8-maintenance-backups--disk-management)
9. [Troubleshooting Common WSL 2 Issues](#-9-troubleshooting-common-wsl-2-issues)

---

## 🧠 1. Understanding WSL 2 Architecture

WSL 2 runs a **real Linux kernel** inside a lightweight, highly optimized Hyper-V utility virtual machine.

### WSL 1 vs. WSL 2

| Feature | WSL 1 | WSL 2 |
| :--- | :--- | :--- |
| **Architecture** | System call translation layer (translates Linux syscalls to Windows NT APIs). | Real Linux kernel running in a micro-VM. |
| **Linux Kernel** | None (emulated). | Real Linux kernel maintained and updated by Microsoft. |
| **System Call Compatibility** | Partial (fails on complex syscalls like Docker, eBPF, FUSE). | **100% full system call compatibility**. |
| **Systemd Support** | No | **Yes** (supported natively). |
| **Cross-OS I/O Speed** | Faster on `/mnt/c/` | Slower on `/mnt/c/` (via 9P protocol), but **blazing fast** on root ext4 (`/home/`). |
| **Docker Support** | Requires third-party emulation hacks. | Native Docker Engine or Docker Desktop integration. |
| **GPU Acceleration** | Not supported. | Native Direct3D 12 and NVIDIA CUDA passthrough. |

---

## 🚀 2. Prerequisites & Installation

### Step 1: Enable Hardware Virtualization in BIOS/UEFI

Make sure **Intel Virtualization Technology (VT-x)** or **AMD-V (SVM Mode)** is enabled in your computer's BIOS/UEFI settings.

### Step 2: One-Command Installation (Run in PowerShell as Administrator)

Open **PowerShell** or **Windows Terminal** as Administrator:

```powershell
# Installs WSL 2, the Virtual Machine Platform, and downloads default Ubuntu
wsl --install
```

To list all available Linux distributions:

```powershell
wsl --list --online
```

To install a specific distribution (e.g., Ubuntu 24.04 or Debian):

```powershell
wsl --install -d Ubuntu-24.04
# or
wsl --install -d Debian
```

### Step 3: Set Default WSL Version & Verify

```powershell
# Ensure version 2 is always used for new distros
wsl --set-default-version 2

# Update the Linux kernel to the latest release
wsl --update

# Verify currently installed distributions and their version
wsl --list --verbose
# (Shortcut: wsl -l -v)
```

Expected output:

```text
  NAME            STATE           VERSION
* Ubuntu-24.04    Running         2
```

---

## ⚙️ 3. Core Configuration Files

Tuning WSL 2 requires configuring two separate configuration files:

1. **`.wslconfig`**: Applies globally to **all** WSL 2 distributions (configured in Windows).
2. **`/etc/wsl.conf`**: Applies to a **single** specific Linux distribution (configured inside Linux).

---

### 1. Host-wide Configuration: `.wslconfig`

By default, WSL 2 can consume up to 50% (or more) of your total Windows RAM, causing your Windows host to freeze under heavy workloads.

Create or edit this file in Windows at:
`C:\Users\<YourWindowsUsername>\.wslconfig`

```ini
[wsl2]
# Limit VM memory allocation (e.g., half of total system RAM)
memory=8GB

# Limit VM CPU cores
processors=4

# Configure swap space on Windows SSD
swap=4GB
swapFile=C:\\Temp\\wsl-swap.vhdx

# Enable localhost forwarding (access Linux web servers via localhost in Windows browsers)
localhostForwarding=true

# (Windows 11 / WSL 2.0+ Features)
[experimental]
# Automatically releases cached memory back to Windows
autoMemoryReclaim=gradual

# Mirrored networking improves VPN compatibility and connection speed
networkingMode=mirrored

# Automatically shrinks unused ext4 virtual disk space
sparseVhd=true
```

[!IMPORTANT]
Whenever you modify `.wslconfig`, you **must restart WSL** for changes to take effect:

```powershell
wsl --shutdown
```

---

### 2. Distro-level Configuration: `/etc/wsl.conf`

Open a terminal inside your Linux distro (`Ubuntu`) and edit `/etc/wsl.conf`:

```bash
sudo nano /etc/wsl.conf
```

Paste the following production configuration:

```ini
[boot]
# Enable systemd so services like Docker, cron, and ssh can run automatically
systemd=true

[user]
# Set default login user (replace with your Linux username)
default=ahmad

[network]
# Set hostname for this distro
hostname=dev-wsl

# Uncomment if you want to manage /etc/resolv.conf manually for custom DNS/VPNs
# generateResolvConf = false

[automount]
# Automatically mount Windows drives (C:\, D:\) under /mnt/
enabled=true
mountFsTab=true
root=/mnt/
# Enforce correct Linux file permissions on Windows mounts
options="metadata,umask=22,fmask=11"

[interop]
# Allow launching Windows binaries (.exe) from the Linux terminal
enabled=true
appendWindowsPath=true
```

After editing, restart WSL from PowerShell:

```powershell
wsl --shutdown
```

Verify that `systemd` is running:

```bash
systemctl is-system-running
# Should output: running (or degraded due to optional services)
```

---

## ⚡ 4. Filesystem Performance: The Golden Rule

Understanding cross-OS filesystem boundaries is the single most important factor for WSL 2 performance.

```text
+----------------------------------------------------------------------+
|                           WINDOWS HOST (NTFS)                        |
|   Path: C:\Users\name\Projects                                       |
|   Speed from Windows: 🚀 BLAZING FAST                                |
|   Speed from WSL 2:   🐢 VERY SLOW (Crosses 9P Network Bridge)       |
+----------------------------------------------------------------------+
                                   ▲
                                   │ (Cross-boundary penalty: 10x-50x slower)
                                   ▼
+----------------------------------------------------------------------+
|                          WSL 2 DISTRO (ext4)                         |
|   Path: /home/username/projects                                      |
|   Speed from WSL 2:   🚀 NATIVE LINUX BARE-METAL SPEED               |
|   Speed from Windows: ⚡ FAST (via \\wsl$\ virtual network share)    |
+----------------------------------------------------------------------+
```

### The Rules

1. **Always store your code and projects inside Linux**:
   - ✅ **CORRECT**: `/home/username/projects/my-app`
   - ❌ **INCORRECT**: `/mnt/c/Users/YourName/projects/my-app`
2. **Never run `npm install`, `pip install`, `git status`, or `cargo build` on `/mnt/c/`**:
   - Operations that access thousands of small files will be **10x to 50x slower** on Windows mounts due to 9P translation overhead.

### Accessing Files Across Boundaries Seamlessly

* **From Linux to Windows**:

  Windows drives are automatically mounted:
  
  ```bash
  cd /mnt/c/Users/<WindowsUsername>/Downloads
  ```

* **From Windows to Linux**:
  Access the ext4 filesystem directly in Windows File Explorer:
  
  ```text
  \\wsl$\Ubuntu\home\<username>
  ```

  Or from inside your WSL terminal, open File Explorer directly in the current directory:
  
  ```bash
  explorer.exe .
  ```

---

## 🛠️ 5. Development Tooling Integration

### VS Code (Remote - WSL)

The official **Remote - WSL** extension allows VS Code to run its UI on Windows while running the language server, terminal, debugger, and extensions directly inside Linux.

1. Install **VS Code** on Windows.
2. Install the **WSL** extension (`ms-vscode-remote.remote-wsl`) from the Extensions Marketplace.
3. Inside your WSL terminal, navigate to any project folder and type:

   ```bash
   code .
   ```

   VS Code will download the lightweight WSL server component and attach seamlessly.

---

### Git & SSH Credential Sharing

#### Method A: Linux-Native SSH Keys (Recommended)

Generate and keep SSH keys securely inside Linux:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

#### Method B: Use Windows Git Credential Manager (GCM)

Share HTTPS GitHub credentials stored in Windows with Linux:

```bash
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

---

### Windows Terminal & Nerd Fonts

1. Install **Windows Terminal** from Microsoft Store.
2. Install a **Nerd Font** (e.g., *MesloLGS NF*, *JetBrainsMono Nerd Font*, or *FiraCode Nerd Font*) on Windows to enable modern prompt icons and glyphs.
3. In Windows Terminal Settings (`Ctrl + ,`):
   * Set **Default Profile** to **Ubuntu**.
   * Under **Profiles > Ubuntu > Appearance > Font Face**, select your installed Nerd Font.

---

## 🐳 6. Docker in WSL 2: Two Approaches

You have two distinct methods to run Docker on WSL 2:

### Approach A: Native Docker Engine (Lightweight & Free)

With `systemd` enabled in `/etc/wsl.conf`, you can install the official Docker Engine directly inside Ubuntu **without needing Docker Desktop**.

* **Pros**: Uses significantly fewer Windows resources; completely free for commercial use.
* **Cons**: No graphical dashboard.

```bash
# Follow the official Docker guide:
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
sudo systemctl enable --now docker
```

---

### Approach B: Docker Desktop with WSL 2 Backend

* **Pros**: Graphical UI, Kubernetes support, automatic resource integration.
* **Installation**:
  1. Install Docker Desktop for Windows.
  2. In Docker Desktop Settings:
     * **General**: Check *Use the WSL 2 based engine*.
     * **Resources > WSL Integration**: Enable integration with your specific distro (e.g. *Ubuntu*).

---

## 🎮 7. GPU Acceleration (NVIDIA CUDA Setup)

WSL 2 natively supports direct GPU passthrough for machine learning frameworks (PyTorch, TensorFlow) and graphics compute.

### Rules for CUDA on WSL 2

1. **Install NVIDIA Drivers on Windows ONLY**:
   * Download and install the latest Game Ready or Studio Driver on Windows.
   * **DO NOT** install Linux display drivers (`nvidia-driver-xxx`) inside WSL! Doing so will break the WSL driver passthrough bridge.
2. **Inside WSL 2, install the CUDA Toolkit only**:

```bash
# 1. Verify GPU is detected by WSL
nvidia-smi
```

Expected output:

```text
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.xx                 Driver Version: 550.xx         CUDA Version: 12.4     |
|   0  NVIDIA GeForce RTX ...                         ...                                 |
+-----------------------------------------------------------------------------------------+
```

```bash
# 2. Install CUDA Toolkit for WSL-Ubuntu (Example for Ubuntu 22.04 / 24.04):
sudo apt install -y nvidia-cuda-toolkit
```

---

## 💾 8. Maintenance, Backups & Disk Management

### Lifecycle Commands (Run in Windows PowerShell)

```powershell
# Terminate a single running distribution:
wsl --terminate Ubuntu

# Completely shut down all WSL 2 instances and release host RAM:
wsl --shutdown

# Start a specific distribution as root:
wsl -d Ubuntu -u root
```

---

### Backup and Restore Distros

You can export an entire Linux environment to a `.tar` archive and restore it anytime or migrate it to another PC:

```powershell
# Create a full backup of your distro
wsl --export Ubuntu D:\Backups\ubuntu_backup.tar

# Restore/Import the backup as a new distro
wsl --import Ubuntu-Dev D:\WSL\Ubuntu-Dev D:\Backups\ubuntu_backup.tar --version 2

# Remove/Unregister an existing distro (WARNING: permanently deletes data)
wsl --unregister Ubuntu-Old
```

---

### Reclaiming Disk Space from `ext4.vhdx`

WSL 2 virtual hard drives (`.vhdx`) grow automatically as you write files, but they **do not automatically shrink** when files are deleted inside Linux.

#### Reclaim Space Manually

1. Inside Linux, clean up package caches and unused Docker files:

   ```bash
   sudo apt clean
   docker system prune -a --volumes -f
   ```

2. In Windows PowerShell:

   ```powershell
   # 1. Shut down WSL
   wsl --shutdown

   # 2. Open diskpart tool
   diskpart
   ```

3. Inside the `DISKPART>` prompt:

   ```text
   select vdisk file="C:\Users\<YourUser>\AppData\Local\Packages\<CanonicalGroup...>\LocalState\ext4.vhdx"
   attach vdisk readonly
   compact vdisk
   detach vdisk
   exit
   ```

*(On Windows 11 with `sparseVhd=true` in `.wslconfig`, you can simply run `wsl --manage Ubuntu --compact`)*.

---

## 🩺 9. Troubleshooting Common WSL 2 Issues

### Issue 1: "Wsl/Service/CreateInstance/CreateVm/HCS/E_INVALIDARG" or Virtualization Error

* **Cause**: Virtual Machine Platform is disabled or BIOS Virtualization (VT-x / AMD-V) is off.
* **Resolution**:
  In PowerShell as Admin:

  ```powershell
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  ```

  Restart your physical PC and check BIOS virtualization settings.

---

### Issue 2: No Internet Access / DNS Resolution Failure in WSL

* **Cause**: Automatic `/etc/resolv.conf` generation conflicts with corporate VPNs (Cisco AnyConnect, GlobalProtect).
* **Resolution**:
  1. Disable auto-generation in `/etc/wsl.conf`:

     ```ini
     [network]
     generateResolvConf = false
     ```

  2. Remove the symlink and set public DNS manually:

     ```bash
     sudo rm -f /etc/resolv.conf
     sudo bash -c 'echo "nameserver 1.1.1.1" > /etc/resolv.conf'
     sudo bash -c 'echo "nameserver 8.8.8.8" >> /etc/resolv.conf'
     ```

  3. Restart WSL via `wsl --shutdown`.

---

### Issue 3: Line Endings Warning (`CRLF` vs `LF`)

* **Cause**: Windows tools use `\r\n` (CRLF) while Linux uses `\n` (LF). Shell scripts with CRLF fail in Linux with `\r: command not found`.
* **Resolution**:
  Configure Git to preserve Linux line endings inside WSL:

  ```bash
  git config --global core.autocrlf input
  ```

  Convert existing corrupted files using `dos2unix`:

  ```bash
  sudo apt install -y dos2unix
  dos2unix script.sh
  ```

---

## 🎯 Quick Reference Cheat Sheet

| Action | Command / Location | Run From |
| :--- | :--- | :--- |
| **Check WSL Status & Versions** | `wsl -l -v` | PowerShell |
| **Full WSL Shutdown (Free RAM)** | `wsl --shutdown` | PowerShell |
| **Open Current Folder in File Explorer** | `explorer.exe .` | Linux |
| **Open Current Folder in VS Code** | `code .` | Linux |
| **Access Windows C Drive** | `cd /mnt/c/` | Linux |
| **Access Linux from Windows Explorer** | `\\wsl$\<DistroName>\home\<user>` | Windows |
| **Global Host Settings** | `%USERPROFILE%\.wslconfig` | Windows |
| **Distro Settings (systemd, mounts)** | `/etc/wsl.conf` | Linux |
