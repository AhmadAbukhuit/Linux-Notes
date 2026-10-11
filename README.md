# 🐧 Linux Notes & Reference Guide

A curated, living collection of Linux commands, tool configurations, and system administration notes.

Whether you are configuring a fresh Debian server, optimizing a WSL 2 development environment, or just looking up that one `tar` command you always forget, this repository serves as a quick, practical reference.

## 📂 Contents

* **[Overview](./01-overview/)**:
  * [01 - Introduction to Linux](./01-overview/01_introduction.md)
  * [02 - Linux Filesystem Hierarchy](./01-overview/02_filesystem-hierarchy.md)
  * [03 - Linux Permissions & Ownership](./01-overview/03_permissions.md)
* **[Core Commands](./02-core-commands/)**:
  * [01 - Command Reference](./02-core-commands/01_command.md)
  * [02 - Systemd & Service Management](./02-core-commands/02_systemd_and_services.md)
  * [03 - Process & Resource Monitoring](./02-core-commands/03_process_and_resource_monitoring.md)
  * [04 - Bash Scripting & Automation](./02-core-commands/04_bash_scripting_and_automation.md)
* **[Environments](./03-environments/)**:
  * [WSL 2 Setup & Optimization Guide](./03-environments/01_wsl2_setup.md)
* **[Tools & Infrastructure](./04-tools_and_infrastructure/)**:
  * [01 - Git & GitHub Essentials](./04-tools_and_infrastructure/01_git_and_github.md)
  * [02 - Docker & Docker Compose v2 Guide](./04-tools_and_infrastructure/02_docker_and_docker-compose.md)
  * [03 - SSH & Remote Access Mastery](./04-tools_and_infrastructure/03_ssh_and_remote_access.md)
  * [04 - Linux Networking & Firewall (UFW)](./04-tools_and_infrastructure/04_networking_and_firewall.md)

## 🚀 How to Use This Repository

You can browse the folders directly, or clone the repository locally to search through the markdown files using `grep` or your favorite text editor:

```bash
git clone https://github.com/AhmadAbukhuit/Linux-Notes.git
cd linux-notes
grep -r "docker-compose" .
```

## 🤝 Contributing

Contributions, corrections, and additions are welcome! If you have a command or tool explanation that saves time, please open a pull request.

Check out our [Contributing Guidelines](./CONTRIBUTING.md) to get started.

## 📄 License

This project is licensed under the [MIT License](./LICENSE).
