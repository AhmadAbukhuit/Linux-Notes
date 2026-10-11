# 🛠️ Section 04: Tools, Infrastructure & Security

Welcome to the **Tools, Infrastructure & Security** module. This section provides in-depth, production-oriented references for version control, containerization, remote access, networking diagnostics, and server hardening on Linux.

---

## 📑 Contents in this Folder

| File | Title | Summary & Key Topics |
| :--- | :--- | :--- |
| **[`01_git_and_github.md`](./01_git_and_github.md)** | **Git & GitHub Essentials** | Initial configuration, generating Ed25519 SSH keys for GitHub, core workflows (`clone`, `status`, `commit`, `push`, `pull`), branch management, merging, and managing large machine learning datasets and binary models using **Git LFS**. |
| **[`02_docker_and_docker-compose.md`](./02_docker_and_docker-compose.md)** | **Docker & Docker Compose v2 Guide** | Complete Linux installation guide for Docker Engine and Compose v2 (`docker compose` plugin), post-install non-root configuration, Docker CLI reference (images, containers, volumes, networks), multi-stage Dockerfile builds, production `compose.yaml` multi-service orchestration (Web + DB + Redis), and troubleshooting. |
| **[`03_ssh_and_remote_access.md`](./03_ssh_and_remote_access.md)** | **SSH & Remote Access Mastery** | Modern Ed25519 key generation, strict permission rules, client configuration tricks in `~/.ssh/config` (aliases, keepalives, Bastion `ProxyJump`, `ControlMaster` multiplexing), production server hardening in `/etc/ssh/sshd_config`, encrypted tunnels (local/remote port forwarding, SOCKS5 proxy), and high-speed folder sync with `rsync`. |
| **[`04_networking_and_firewall.md`](./04_networking_and_firewall.md)** | **Linux Networking & Firewall (UFW)** | Modern `iproute2` inspection (`ip a`, `ip r`), socket and port discovery with `ss -tulpn`, latency testing with `mtr`, port reachability with `nc -zv`, DNS troubleshooting with `dig` and `systemd-resolved`, firewall configuration using **UFW** (rules, rate limiting, numbered deletions), and brute-force banning with **Fail2ban**. |

---

## 🧭 Infrastructure Workflow Map

```text
+------------------------------------+
|       01_git_and_github.md         |  Version control code & track models via Git LFS
+-----------------+------------------+
                  |
                  v
+------------------------------------+
|   02_docker_and_docker-compose.md  |  Containerize applications & orchestrate microservices
+-----------------+------------------+
                  |
                  v
+------------------------------------+
|    03_ssh_and_remote_access.md     |  Securely access servers, jump hosts & sync with Rsync
+-----------------+------------------+
                  |
                  v
+------------------------------------+
|    04_networking_and_firewall.md   |  Inspect open ports, configure UFW & block attacks with Fail2ban
+------------------------------------+
```

---

## 🚀 Navigation

* Back to **[Main Repository Index](../README.md)**
* Review **[Core Commands & System Administration](../02-core-commands/README.md)**
* Review **[Linux Overview & Permissions](../01-overview/README.md)**
