# ⚙️ Section 02: Core Commands & System Administration

Welcome to the **Core Commands & System Administration** module. This section provides an extensive, production-grade reference for everyday terminal commands, background daemon management, process triage, and automation scripting.

---

## 📑 Contents in this Folder

| File | Title | Summary & Key Topics |
| :--- | :--- | :--- |
| **[`01_command.md`](./01_command.md)** | **Linux Commands Reference** | Everyday command syntax, directory navigation (`pwd`, `cd`, `ls`), file operations (`cp`, `mv`, `rm`, `find`), viewing file content (`cat`, `less`, `head`, `tail`), compression (`tar`, `zip`, `7z`), user management, and text filtering (`grep`, `awk`, `sed`, `cut`). |
| **[`02_systemd_and_services.md`](./02_systemd_and_services.md)** | **Systemd & Service Management** | Managing services with `systemctl` (`start`, `stop`, `enable --now`, `status`, `mask`), writing custom systemd `.service` units (with user isolation, `.env` files, and auto-restart), querying logs with `journalctl` (`-f`, `-u`, `--since`), disk vacuuming, and replacing Cron with systemd timers. |
| **[`03_process_and_resource_monitoring.md`](./03_process_and_resource_monitoring.md)** | **Process & Resource Monitoring** | Linux process lifecycle and states (`R`, `S`, `D`, `Z`), `ps` sorting recipes, interactive monitoring (`top`, `htop`, `btop`), process signals (`SIGTERM`, `SIGKILL`, `SIGHUP`), background tasks (`&`, `nohup`, `disown`), memory analysis (`free -h`, page cache, OOM Killer), and disk I/O bottlenecks (`iotop`, `ncdu`). |
| **[`04_bash_scripting_and_automation.md`](./04_bash_scripting_and_automation.md)** | **Bash Scripting & Automation** | Writing bulletproof shell scripts using strict mode (`set -euo pipefail`), parameter expansion, standard streams & file descriptors (`2>&1`, `tee`, heredocs), modern `[[ ... ]]` conditionals, argument parsing with `getopts`, cleanup `trap` handlers, concurrency locks (`flock`), and Cron scheduling. |

---

## 🧭 Learning Path

```text
+------------------------------------+
|          01_command.md             |  Master terminal navigation, file manipulation & text filtering
+-----------------+------------------+
                  |
                  v
+------------------------------------+
|  03_process_and_resource_monitoring |  Inspect CPU/RAM/IO, triage bottlenecks & manage background jobs
+-----------------+------------------+
                  |
                  v
+------------------------------------+
|    02_systemd_and_services.md      |  Run production background daemons, timers & journal logs
+-----------------+------------------+
                  |
                  v
+------------------------------------+
| 04_bash_scripting_and_automation   |  Automate tasks defensively with Bash scripts & Cron
+------------------------------------+
```

---

## 🚀 Next Steps

Once you master core commands and service administration:

* Explore virtualized environments in **[03-environments](../03-environments/README.md)** (WSL 2 setup and performance optimization).
* Learn modern developer infrastructure in **[04-tools_and_infrastructure](../04-tools_and_infrastructure/README.md)** (Git, Docker, SSH, and Firewall security).
