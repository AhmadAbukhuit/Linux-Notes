# 💻 Section 03: Linux Environments & Setups

Welcome to the **Linux Environments & Setups** module. This section provides detailed setup guides, hardware virtualization configuration, and tuning instructions for running Linux across local workstations, virtual machines, and hybrid development platforms.

---

## 📑 Contents in this Folder

| File | Title | Summary & Key Topics |
| :--- | :--- | :--- |
| **[`01_wsl2_setup.md`](./01_wsl2_setup.md)** | **WSL 2 Setup & Optimization Guide** | Complete guide for Windows Subsystem for Linux 2. Architecture (micro-VM vs translation layer), one-command install, host-wide RAM limits in `.wslconfig`, distro-level `systemd` in `/etc/wsl.conf`, the **cross-OS filesystem performance rule** (avoiding slow `/mnt/c/`), VS Code Remote-WSL integration, Docker setup, NVIDIA CUDA GPU passthrough for ML, and compacting virtual disks (`.vhdx`). |

---

## 🌟 Key Principles for Virtualized Environments

When running Linux in virtualized or hybrid environments (like WSL 2, Hyper-V, or VirtualBox):

1. **Storage Performance Boundary**: Always store repositories and source code inside the native Linux virtual disk (`/home/<user>/`), never on mounted host drives (like `/mnt/c/`).
2. **Resource Throttling**: Configure explicit memory and CPU core ceilings (such as `.wslconfig`) to prevent virtual machines from starving the host operating system.
3. **Init System Integration**: Enable `systemd` to allow modern container runtimes (Docker, Podman) and services to operate identically to bare-metal servers.

---

## 🚀 Next Steps

* Move on to **[04-tools_and_infrastructure](../04-tools_and_infrastructure/README.md)** to configure developer tooling, containers, and network security.
* Refer back to **[02-core-commands](../02-core-commands/README.md)** for system administration and shell scripting techniques.
