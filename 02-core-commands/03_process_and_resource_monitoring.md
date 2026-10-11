# Process & Resource Monitoring Guide

A comprehensive, production-oriented reference for inspecting Linux processes, managing background jobs, analyzing CPU and memory consumption, troubleshooting disk I/O bottlenecks, and responding to system resource emergencies.

---

## 📑 Table of Contents

1. [Understanding Linux Processes](#-1-understanding-linux-processes)
   - [Process States & Lifecycle](#process-states--lifecycle)
2. [Process Inspection & Monitoring Tools](#-2-process-inspection--monitoring-tools)
   - [Standard `ps` Mastery](#standard-ps-mastery)
   - [Interactive Monitoring: `top`, `htop`, and `btop`](#interactive-monitoring-top-htop-and-btop)
   - [Process Tree Visualization (`pstree`)](#process-tree-visualization-pstree)
3. [Signals & Terminating Processes](#-3-signals--terminating-processes)
   - [Key POSIX Signals](#key-posix-signals)
   - [`kill`, `killall`, and `pkill`](#kill-killall-and-pkill-commands)
4. [Job Control & Background Tasks](#️-4-job-control--background-tasks)
   - [Backgrounding Processes (`&`, `fg`, `bg`, `jobs`)](#backgrounding-processes)
   - [Persistent Execution (`nohup`, `disown`)](#persistent-execution-nohup--disown)
5. [Memory Triage & Analysis](#-5-memory-triage--analysis)
   - [Understanding `free -h` & Linux Page Cache](#understanding-free--h--linux-page-cache)
   - [The OOM (Out Of Memory) Killer](#the-oom-out-of-memory-killer)
6. [Disk Space & Storage I/O Bottlenecks](#-6-disk-space--storage-io-bottlenecks)
   - [Disk Space & Inode Inspection (`df`, `du`, `ncdu`)](#disk-space--inode-inspection)
   - [Disk I/O Analysis (`iostat`, `iotop`)](#disk-io-analysis-iostat-iotop)
7. [Emergency Triage Playbooks](#-7-emergency-triage-playbooks)
8. [Quick Reference Cheat Sheet](#-8-quick-reference-cheat-sheet)

---

## 🧬 1. Understanding Linux Processes

In Linux, a **process** is an active instance of a running program. Every process is assigned a unique integer identifier called a **PID** (Process ID).

* **PID 1**: The init system (`systemd`), ancestor of all other user-space processes.
* **PPID**: Parent Process ID (the process that spawned this child process).
* **/proc Filesystem**: The kernel exposes real-time process metadata through the virtual `/proc` directory (e.g., `/proc/<PID>/status`, `/proc/<PID>/cmdline`).

### Process States & Lifecycle

| Code | State | Description |
| :---: | :--- | :--- |
| **R** | **Running / Runnable** | Actively executing on CPU or waiting in the run queue. |
| **S** | **Interruptible Sleep** | Waiting for an event (I/O, timer, user input). Can be awakened by signals. |
| **D** | **Uninterruptible Sleep** | Usually waiting on disk/NFS I/O or kernel lock. **Cannot be killed with `kill -9`** until I/O completes. |
| **Z** | **Zombie / Defunct** | Terminated process whose parent has not yet read its exit status via `wait()`. Consumes no CPU/RAM, only a PID slot. |
| **T** | **Stopped / Traced** | Suspended by job control signal (`Ctrl + Z` / `SIGTSTP`) or a debugger (`gdb`). |

---

## 🔍 2. Process Inspection & Monitoring Tools

### Standard `ps` Mastery

`ps` reports a static snapshot of the current processes.

#### Standard Flags

* `ps aux`: BSD style (most popular).
  * `a`: Show processes for all users.
  * `u`: Display user/owner-oriented format (CPU%, MEM%).
  * `x`: Include processes not attached to a terminal (daemons).
* `ps -ef`: Standard POSIX / System V style (shows UID, PID, PPID, C, STIME, TTY, TIME, CMD).

#### Powerful `ps` Snippets for Production Triage

```bash
# 1. Top 10 CPU-consuming processes:
ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu | head -n 11

# 2. Top 10 Memory-consuming processes:
ps -eo pid,ppid,user,%mem,%cpu,cmd --sort=-%mem | head -n 11

# 3. Find all processes belonging to a specific user:
ps -u www-data -o pid,stat,%cpu,%mem,cmd

# 4. Search for a specific process name with arguments:
ps aux | grep "[n]ginx"
# (Pro Tip: Using [n] prevents grep from finding its own process!)
```

---

### Interactive Monitoring: `top`, `htop`, and `btop`

#### Standard `top`

Every Linux server includes `top` out of the box.

```bash
top
```

**Essential In-Tool Keyboard Shortcuts in `top`:**

* `M` (Shift + M): Sort processes by **Memory usage** (descending).
* `P` (Shift + P): Sort processes by **CPU usage** (descending).
* `T` (Shift + T): Sort processes by **Running Time**.
* `k`: Kill a process (prompts for PID and signal number).
* `c`: Toggle display of full command-line path and arguments.
* `1`: Toggle breakdown of individual CPU cores.
* `q`: Quit.

---

#### Modern Alternative: `htop`

`htop` provides an intuitive, color-coded, scrollable UI with visual meters for CPU cores, RAM, and swap.

```bash
# Install on Ubuntu/Debian:
sudo apt install -y htop

# Install on RHEL/CentOS:
sudo dnf install -y epel-release && sudo dnf install -y htop

# Launch:
htop
```

* `F3`: Search processes.
* `F4`: Incremental filter (type name to filter in real-time).
* `F5`: Tree view (displays parent-child relationships).
* `F6`: Select sort column (`%CPU`, `%MEM`, `TIME`).
* `F9`: Send kill signal directly from UI.

---

#### Modern Terminal Dashboard: `btop`

Written in C++, `btop` offers a visually stunning dashboard including network traffic, disk I/O, process trees, and GPU monitoring.

```bash
sudo apt install -y btop   # Ubuntu 22.04+
btop
```

---

### Process Tree Visualization (`pstree`)

Understand parent-child process hierarchies and find who spawned a rogue process:

```bash
# Show process tree with PIDs
pstree -p

# Show tree for a specific parent PID
pstree -p 1234

# Highlight currently running processes
pstree -h
```

---

## 🛑 3. Signals & Terminating Processes

Signals are asynchronous notifications sent to processes by the kernel or users.

### Key POSIX Signals

| Signal Name | Number | Catchable? | Purpose / Behavior |
| :--- | :---: | :---: | :--- |
| **`SIGTERM`** | `15` | **Yes** | **Graceful termination** (Default). Asks process to clean up temporary files, flush buffers, and exit cleanly. |
| **`SIGKILL`** | `9` | **No** | **Forced termination**. Handled directly by kernel; process cannot intercept or ignore it. Can corrupt uncommitted files. |
| **`SIGINT`** | `2` | **Yes** | **Interrupt from keyboard** (`Ctrl + C`). |
| **`SIGQUIT`** | `3` | **Yes** | Quit from keyboard (`Ctrl + \`), dumps core memory file. |
| **`SIGHUP`** | `1` | **Yes** | **Hangup**. Often used by daemons (like Nginx/sshd) to **reload configuration** without restarting. |
| **`SIGSTOP`** | `19` | **No** | Pauses/freezes process execution immediately. |
| **`SIGCONT`** | `18` | **Yes** | Resumes a previously paused process (`SIGSTOP`). |

---

### `kill`, `killall`, and `pkill` Commands

#### 1. `kill`: Terminate by PID

Always attempt graceful termination (`SIGTERM`) first before resorting to `SIGKILL` (`-9`):

```bash
# Graceful stop (sends SIGTERM / 15):
kill 2345

# Wait 5 seconds, and if it refuses to terminate, force kill:
kill -9 2345
# (or: kill -s SIGKILL 2345)

# Reload daemon configuration without dropping connections:
kill -HUP $(cat /var/run/nginx.pid)
```

#### 2. `pkill` and `pgrep`: Match by Name or Regex

Search and kill processes based on pattern matching:

```bash
# Find PIDs matching a process name:
pgrep -l python

# Kill processes matching pattern:
pkill -f "celery worker"

# Kill all processes belonging to a specific user:
sudo pkill -u developer
```

#### 3. `killall`: Exact Name Match

```bash
killall nginx
killall -9 php-fpm
```

---

## ⚙️ 4. Job Control & Background Tasks

### Backgrounding Processes

| Command / Key | Action |
| :--- | :--- |
| `command &` | Launches command directly in the background. |
| `Ctrl + Z` | Suspends (pauses) the currently running foreground process (`SIGTSTP`). |
| `jobs` | Lists all background and suspended jobs in current shell. |
| `bg %1` | Resumes job #1 in the background. |
| `fg %1` | Brings job #1 back to the foreground. |

```bash
# Example Workflow:
# 1. Start a long-running compression task:
tar -czf backup.tar.gz /var/data/

# 2. You realize it takes too long; press:
[Ctrl + Z]  # Suspends the job: [1]+  Stopped

# 3. Resume it in the background:
bg %1       # [1]+ tar -czf backup.tar.gz /var/data/ &

# 4. Check status:
jobs -l
```

---

### Persistent Execution (`nohup` & `disown`)

By default, when an SSH session disconnects, the shell sends a `SIGHUP` signal to all child processes, terminating them.

#### Option A: `nohup` (No Hang Up)

Ignores `SIGHUP` and redirects standard output and error to a log file:

```bash
nohup python3 script.py > script.log 2>&1 &
```

* `> script.log`: Redirects stdout to `script.log`.
* `2>&1`: Redirects stderr to the same file.
* `&`: Sends process to background.

#### Option B: `disown` (Detach Existing Running Job)

If you already started a process and forgot to use `nohup`:

```bash
# 1. Suspend it:
[Ctrl + Z]

# 2. Resume in background:
bg

# 3. Detach from current shell session:
disown -h %1
```

Now you can safely close your terminal or disconnect SSH without killing the process!

*(For full terminal multiplexing with interactive reconnects, use **tmux**)*.

---

## 🧠 5. Memory Triage & Analysis

### Understanding `free -h` & Linux Page Cache

```bash
free -h
```

Output example:

```text
               total        used        free      shared  buff/cache   available
Mem:            15Gi       4.2Gi       1.1Gi       210Mi        10Gi        11Gi
Swap:          4.0Gi       120Mi       3.8Gi
```

#### Key Column Breakdown

* **`total`**: Physical installed RAM.
* **`used`**: RAM currently allocated to active processes (`total - free - buff/cache`).
* **`free`**: Unused memory (completely idle).
* **`buff/cache`**: Memory used by the Linux kernel for filesystem cache (read caching) and buffers (write caching).
* **`available`**: **The most important metric!** The amount of memory immediately available for starting new applications **without swapping**. It includes `free` memory plus reclaimable page cache.

> [!NOTE]
> **"Linux ate my RAM!"**: Linux utilizes unused memory to cache files from disk (`buff/cache`), which makes file operations dramatically faster. As soon as applications need that RAM, the kernel releases the cache instantly. High cache usage is healthy!

---

### The OOM (Out Of Memory) Killer

When available RAM and swap space are completely exhausted, the Linux kernel invokes the **OOM Killer** to select and terminate high-memory processes to prevent a catastrophic system panic.

```bash
# Check if the OOM Killer has terminated any process:
sudo dmesg -T | grep -i -E "oom|out of memory|killed process"
# or
journalctl -k --since "24 hours ago" | grep -i oom
```

#### Checking Process OOM Score

The kernel assigns an `oom_score` to each process (0 to 1000). The higher the score, the more likely it is to be killed:

```bash
cat /proc/<PID>/oom_score
```

---

## 💽 6. Disk Space & Storage I/O Bottlenecks

### Disk Space & Inode Inspection

```bash
# 1. Check disk capacity on all mounted filesystems:
df -h

# 2. Check inode usage (running out of inodes prevents creating files even if GBs remain!):
df -i

# 3. Find top 10 largest folders in current directory:
sudo du -ah --max-depth=1 . | sort -hr | head -n 10
```

#### Interactive Disk Visualizer: `ncdu`

```bash
sudo apt install -y ncdu
ncdu /var/log
```

Allows navigating disk consumption with arrow keys and deleting bloated files directly.

---

### Disk I/O Analysis (`iostat`, `iotop`)

If CPU usage is low but the server feels sluggish, the bottleneck is often **disk I/O wait (`%iowait`)**.

```bash
# 1. Monitor real-time I/O statistics (part of sysstat package):
iostat -xz 1 5
```

* Look at **`%util`**: Device saturation percentage (> 80% means disk bottleneck).
* Look at **`await`**: Average time (in ms) for I/O requests.

```bash
# 2. Identify which process is writing/reading heavily to disk:
sudo iotop -o
```

* `-o`: Shows only processes actively performing I/O operations.

---

## 🚨 7. Emergency Triage Playbooks

When a server is unresponsive, follow this sequential playbook:

```text
[System Unresponsive]
       │
       ├──► 1. Check Load & CPU:  $ uptime  &&  $ top -b -n 1 | head -n 20
       │       └── If %wa is high: Disk I/O bottleneck -> check $ sudo iotop -o
       │       └── If %us is high: User app spike -> sort by CPU ($ ps -eo pid,%cpu,cmd --sort=-%cpu)
       │
       ├──► 2. Check Memory:      $ free -h
       │       └── If 'available' < 100MB: Memory leak -> check OOM logs ($ dmesg -T | grep -i oom)
       │
       ├──► 3. Check Disk Space:  $ df -h  &&  $ df -i
       │       └── If / is 100% full: $ sudo du -sh /var/log/* | sort -hr
       │
       └──► 4. Check Zombie Processes: $ ps aux | awk '$8 ~ /Z/ {print $0}'
```

---

## 🎯 8. Quick Reference Cheat Sheet

| Task | Command |
| :--- | :--- |
| **Top 5 CPU Processes** | `ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu \| head -n 6` |
| **Top 5 Memory Processes** | `ps -eo pid,user,%mem,%cpu,cmd --sort=-%mem \| head -n 6` |
| **Graceful Process Termination** | `kill <PID>` (or `kill -15 <PID>`) |
| **Forced Immediate Kill** | `kill -9 <PID>` |
| **Kill Process by Exact Name** | `killall <name>` |
| **Kill by Pattern Search** | `pkill -f "<pattern>"` |
| **Find Process PID by Name** | `pgrep -l "<name>"` |
| **Run Background Persistent Task** | `nohup ./app.sh > app.log 2>&1 &` |
| **Show Real Available RAM** | `free -h` |
| **Interactive Disk Usage Browser** | `ncdu /` |
| **Real-Time Disk I/O by Process** | `sudo iotop -o` |
| **Review OOM Killer Events** | `sudo dmesg -T \| grep -i oom` |
