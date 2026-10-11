# Systemd & Service Management Guide

A comprehensive, production-grade guide to managing system services, background daemons, system logs, and scheduled tasks using **systemd** on Linux.

---

## 📑 Table of Contents

1. [Understanding Systemd Architecture](#️-1-understanding-systemd-architecture)
2. [Managing Services with `systemctl`](#️-2-managing-services-with-systemctl)
3. [Inspecting Service States & Properties](#-3-inspecting-service-states--properties)
4. [Writing Custom Systemd Services](#️-4-writing-custom-systemd-services)
   - [Production Web App Service Example](#production-example-python--fastapi-service)
   - [Unit File Directives Explained](#unit-file-sections-breakdown)
5. [Managing and Reading Logs with `journalctl`](#-5-managing-and-reading-logs-with-journalctl)
   - [Real-time Log Streaming](#real-time-and-filtered-log-inspection)
   - [Log Retention & Disk Vacuuming](#managing-journal-disk-space)
6. [Systemd Timers: The Modern Alternative to Cron](#️-6-systemd-timers-the-modern-alternative-to-cron)
7. [System Targets & Power Management](#-7-system-targets--power-management)
8. [Troubleshooting Failed Services](#-8-troubleshooting-failed-services)

---

## 🏛️ 1. Understanding Systemd Architecture

**systemd** is the standard init system and service manager for modern Linux distributions (Debian, Ubuntu, RHEL, CentOS, Fedora, Arch, and WSL 2).

* It runs as **PID 1** (the very first process executed by the Linux kernel after booting).
* It is responsible for bringing up user space, initializing hardware, mounting filesystems, and managing background daemons.

### Key Unit Types in Systemd

| Unit Extension | Purpose | Example |
| :--- | :--- | :--- |
| `.service` | Background daemons, applications, and web servers. | `nginx.service`, `docker.service` |
| `.timer` | Scheduled jobs controlled by systemd (modern cron). | `logrotate.timer` |
| `.socket` | IPC or network sockets for on-demand socket activation. | `docker.socket` |
| `.mount` | Filesystem mount points (similar to `/etc/fstab`). | `tmp.mount` |
| `.target` | Groups of units representing system states/runlevels. | `multi-user.target`, `graphical.target` |

---

## ⚙️ 2. Managing Services with `systemctl`

`systemctl` is the primary CLI utility for controlling the state of systemd units.

### Service Lifecycle Commands

| Action | Command | Explanation |
| :--- | :--- | :--- |
| **Start** | `sudo systemctl start <service>` | Starts an inactive service immediately. |
| **Stop** | `sudo systemctl stop <service>` | Gracefully stops a running service (`SIGTERM`). |
| **Restart** | `sudo systemctl restart <service>` | Stops then starts the service (used after config changes). |
| **Reload** | `sudo systemctl reload <service>` | Reloads configuration without terminating connections. |
| **Status** | `systemctl status <service>` | Displays active state, PID, memory, uptime, and recent logs. |
| **Enable** | `sudo systemctl enable <service>` | Configures service to start automatically on system boot. |
| **Disable** | `sudo systemctl disable <service>` | Prevents service from starting automatically on boot. |
| **Enable & Start** | `sudo systemctl enable --now <service>` | Enables for boot **and** starts it immediately in one step. |
| **Mask** | `sudo systemctl mask <service>` | Links unit to `/dev/null`, rendering it impossible to start manually or automatically. |
| **Unmask** | `sudo systemctl unmask <service>` | Restores a masked service to normal operation. |

```bash
# Practical Example: Nginx
sudo systemctl start nginx
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

---

## 🔍 3. Inspecting Service States & Properties

Instead of parsing verbose status output, use programmatic query commands:

```bash
# Check if a service is actively running (returns exit code 0 if active):
systemctl is-active docker

# Check if a service is configured to boot automatically:
systemctl is-enabled docker

# Check if a service has failed:
systemctl is-failed nginx

# List all currently running services:
systemctl list-units --type=service --state=running

# List all failed units on the system:
systemctl --failed

# Show detailed low-level properties (memory limit, restart count, main PID):
systemctl show docker --property=MainPID,MemoryCurrent,ActiveState
```

---

## ✍️ 4. Writing Custom Systemd Services

System unit files are stored in two primary locations:

* `/usr/lib/systemd/system/`: Package-installed unit files (do not edit directly).
* `/etc/systemd/system/`: **Administrator-created and override unit files** (always place custom services here).

---

### Production Example: Python / FastAPI Service

Create a service file at `/etc/systemd/system/fastapi-app.service`:

```bash
sudo nano /etc/systemd/system/fastapi-app.service
```

```ini
[Unit]
Description=FastAPI Production Backend API
Documentation=https://api.example.com/docs
After=network.target postgresql.service
Wants=postgresql.service

[Service]
# User and Group to run under (never run web apps as root!)
User=www-data
Group=www-data

# Working directory and execution command
WorkingDirectory=/var/www/my-api
ExecStart=/var/www/my-api/.venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000 --workers 4
ExecReload=/bin/kill -s HUP $MAINPID

# Environment Variables
Environment="PYTHONUNBUFFERED=1"
EnvironmentFile=/var/www/my-api/.env

# Restart Policies (Auto-healing)
Restart=always
RestartSec=5s
StartLimitIntervalSec=60s
StartLimitBurst=5

# Hard Resource & Security Limits
LimitNOFILE=65536
MemoryMax=1G
CPUQuota=150%
NoNewPrivileges=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

---

### Unit File Sections Breakdown

#### `[Unit]` Section: Metadata and Dependencies

* `Description`: Human-readable summary displayed in `status` and system logs.
* `After=`: Ensures this service only starts *after* the specified services/targets are ready (e.g. `network.target`).
* `Requires=`: Strict dependency; if the required unit fails, this service stops.
* `Wants=`: Soft dependency; tries to start the dependent unit, but won't fail if it is unavailable.

#### `[Service]` Section: Execution and Lifecycle

* `User=` / `Group=`: Isolates privileges by running under an unprivileged user (e.g., `www-data` or your app user).
* `WorkingDirectory=`: Changes directory prior to executing `ExecStart`.
* `ExecStart=`: Full absolute path to binary and arguments. **Relative paths are not allowed**.
* `Restart=always`: Automatically restarts the process if it crashes or is killed by an unhandled signal.
* `RestartSec=5s`: Waits 5 seconds before attempting a restart.
* `EnvironmentFile=`: Safely injects sensitive configuration from a restricted `.env` file (e.g. `chmod 600 .env`).

#### `[Install]` Section: Activation Target

* `WantedBy=multi-user.target`: Designates this service to start in the standard multi-user runlevel (normal server state with network enabled).

---

### Activating the New Service

Whenever you create or modify any unit file inside `/etc/systemd/system/`, reload the systemd configuration cache:

```bash
# 1. Reload systemd daemon
sudo systemctl daemon-reload

# 2. Enable and start your new service
sudo systemctl enable --now fastapi-app.service

# 3. Check service health
systemctl status fastapi-app.service
```

---

## 📜 5. Managing and Reading Logs with `journalctl`

`journalctl` queries the centralized, binary `systemd-journald` log manager.

### Real-Time and Filtered Log Inspection

```bash
# Follow logs in real time for a specific service (like tail -f):
journalctl -u fastapi-app -f

# View the last 100 lines for a service:
journalctl -u fastapi-app -n 100

# Follow logs with full un-truncated lines:
journalctl -u fastapi-app -f --no-pager

# Filter logs by time window:
journalctl -u fastapi-app --since "1 hour ago"
journalctl -u fastapi-app --since "2026-10-11 12:00:00" --until "2026-10-11 14:00:00"
journalctl --since today

# View logs exclusively from the current system boot:
journalctl -b

# Filter by error priority level (0=emerg, 1=alert, 2=crit, 3=err, 4=warning):
journalctl -p err..emerg

# Combine service and error filtering:
journalctl -u nginx -p err --since "yesterday"
```

---

### Managing Journal Disk Space

Systemd journal logs can consume gigabytes of disk space if unmanaged.

```bash
# Check current disk space consumed by journal logs:
journalctl --disk-usage

# Retain only the last 500 MB of logs (vacuum old records):
sudo journalctl --vacuum-size=500M

# Retain only logs from the last 7 days:
sudo journalctl --vacuum-time=7d
```

#### Permanent Log Retention Configuration

Edit `/etc/systemd/journald.conf`:

```ini
[Journal]
SystemMaxUse=500M
MaxRetentionSec=1month
```

Apply via:

```bash
sudo systemctl restart systemd-journald
```

---

## ⏱️ 6. Systemd Timers: The Modern Alternative to Cron

Systemd timers replace traditional cron jobs with superior logging, dependency tracking, and failure handling.

A timer requires two files with matching base names:

1. `/etc/systemd/system/backup.service` (defines **what** to do)
2. `/etc/systemd/system/backup.timer` (defines **when** to do it)

### Step 1: Create the Service Unit (`backup.service`)

```ini
[Unit]
Description=Database Daily Backup Task

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup-database.sh
User=root
```

### Step 2: Create the Matching Timer Unit (`backup.timer`)

```ini
[Unit]
Description=Run Database Backup Daily at 02:00 AM

[Timer]
# Run every day at 2:00 AM
OnCalendar=*-*-* 02:00:00

# If the system was powered off at 2:00 AM, run immediately upon next boot:
Persistent=true

# Add randomized delay (e.g., 5 min) to prevent server thundering herds
RandomizedDelaySec=300

[Install]
WantedBy=timers.target
```

### Step 3: Enable and View Timers

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer

# View all active system timers and countdown to next trigger:
systemctl list-timers
```

---

## 🎯 7. System Targets & Power Management

Targets represent system states (analogous to legacy SysVinit runlevels).

| Systemd Target | SysVinit Runlevel | State Description |
| :--- | :--- | :--- |
| `poweroff.target` | 0 | Shuts down the system. |
| `rescue.target` | 1 | Single-user administrative repair mode. |
| `multi-user.target` | 3 | Standard headless server environment with networking. |
| `graphical.target` | 5 | Multi-user system with full desktop GUI environment. |
| `reboot.target` | 6 | Reboots the system. |

```bash
# View default boot target:
systemctl get-default

# Set default boot target to headless server:
sudo systemctl set-default multi-user.target

# Power operations:
sudo systemctl poweroff   # Clean shutdown
sudo systemctl reboot     # Clean reboot
```

---

## 🩺 8. Troubleshooting Failed Services

When a service fails to start or crashes, follow this diagnostic checklist:

### Step 1: Check unit status and exit code

```bash
systemctl status <service_name>
```

Look for:

* **`Active: failed`**: Indicates crash or exit.
* **`code=exited, status=203/EXEC`**: Executable path in `ExecStart` does not exist or lacks execute (`+x`) permissions.
* **`code=exited, status=217/USER`**: The user specified in `User=` does not exist on the Linux host.

### Step 2: Read complete diagnostic logs

```bash
journalctl -xeu <service_name>
```

The `-x` flag displays helpful explanatory catalog entries explaining why systemd failed to launch the unit.

### Step 3: Test the `ExecStart` command manually

Switch to the configured user and run the exact command specified in `ExecStart` to see errors directly in your terminal:

```bash
sudo -u www-data /var/www/my-api/.venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
```

This quickly reveals missing dependencies, database connection refusals, or `.env` parsing errors.
