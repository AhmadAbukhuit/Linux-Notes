# Production Bash Scripting & Automation Guide

A comprehensive, production-grade guide to writing defensive, maintainable, and robust **Bash scripts**, handling arguments and error handling, managing file descriptors, and scheduling automated background jobs with **Cron**.

---

## 📑 Table of Contents

1. [Anatomy of a Robust Script](#️-1-anatomy-of-a-robust-script)
   - [Shebang Portability](#shebang-portability)
   - [Strict Mode: `set -euo pipefail`](#strict-mode-set--euo-pipefail)
   - [Safe IFS (Internal Field Separator)](#safe-internal-field-separator)
2. [Variables, Scope & Parameter Expansion](#-2-variables-scope--parameter-expansion)
   - [Local Variables & Constants](#local-variables--constants)
   - [Parameter Expansion Tricks](#parameter-expansion-tricks)
3. [I/O Redirection, Pipes & File Descriptors](#-3-io-redirection-pipes--file-descriptors)
   - [Managing stdout (1) and stderr (2)](#managing-stdout-1-and-stderr-2)
   - [Dual Logging with `tee`](#dual-logging-with-tee)
   - [Heredocs for Template Generation](#heredocs-for-template-generation)
4. [Conditionals & Control Flow](#-4-conditionals--control-flow)
   - [The Modern Test Operator](#the-modern-test-operator---)
   - [File & String Test Flags](#file--string-test-flags)
   - [Loops & Safe Line-by-Line File Processing](#loops--safe-line-by-line-file-processing)
5. [Functions & Exit Codes](#-5-functions--exit-codes)
   - [Function Architecture & Arguments](#function-architecture--arguments)
   - [Capturing Function Output vs. Exit Codes](#capturing-output-vs-exit-codes)
6. [Command-Line Argument Parsing (`getopts`)](#️-6-command-line-argument-parsing-getopts)
7. [Defensive Scripting: Traps & Locks](#️-7-defensive-scripting-traps--locks)
   - [Automatic Cleanup via `trap`](#automatic-cleanup-via-trap)
   - [Preventing Concurrent Runs with `flock`](#preventing-concurrent-runs-with-flock)
8. [Automated Scheduling with Cron](#-8-automated-scheduling-with-cron)
   - [Crontab Syntax & Cheatsheet](#crontab-syntax--cheat-sheet)
   - [Common Cron Gotchas & Mitigations](#common-cron-gotchas--mitigations)
9. [Production Script Template](#-9-production-script-template)
10. [Quick Reference Cheat Sheet](#-10-quick-reference-cheat-sheet)

---

## 🏗️ 1. Anatomy of a Robust Script

### Shebang Portability

Always use `/usr/bin/env bash` rather than hardcoding `/bin/bash`. This ensures compatibility across Linux distributions, macOS, FreeBSD, and custom user environments:

```bash
#!/usr/bin/env bash
```

---

### Strict Mode: `set -euo pipefail`

By default, Bash continues executing subsequent lines even if a command fails, ignores unset variables, and masks errors inside pipelines. **Always include this strict mode line at the top of every script**:

```bash
set -euo pipefail
```

#### What each flag does

* **`-e` (errexit)**: Exit immediately if any command exits with a non-zero status.
* **`-u` (nounset)**: Treat unset variables and parameters as errors and exit immediately (prevents catastrophic disasters like `rm -rf "$UNSET_VAR/*"`).
* **`-o pipefail`**: If any command in a pipeline fails (e.g. `cmd1 | cmd2 | cmd3`), the entire pipeline returns that command's failure exit code instead of returning `cmd3`'s success code.

---

### Safe Internal Field Separator

By default, Bash splits words on spaces, tabs, and newlines. To prevent filenames containing spaces from breaking loops:

```bash
IFS=$'\n\t'
```

---

## 📦 2. Variables, Scope & Parameter Expansion

### Local Variables & Constants

```bash
# Define read-only constants (cannot be modified or unset):
readonly BACKUP_DIR="/var/backups"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Always use 'local' inside functions:
calculate_total() {
    local price="$1"
    local quantity="$2"
    local total=$((price * quantity))
    echo "$total"
}
```

---

### Parameter Expansion Tricks

Native parameter expansion is faster than spawning external tools like `awk`, `sed`, or `cut`:

```bash
# 1. Default values (if UNSET or EMPTY, use default):
PORT="${APP_PORT:-8080}"

# 2. Assign default value (sets variable if unset):
DATABASE_NAME="${DB_NAME:=production_db}"

# 3. Mandatory variable (exits with error message if missing):
API_TOKEN="${SECRET_TOKEN:?Error: SECRET_TOKEN must be defined!}"

# 4. Strip string prefixes (non-greedy # and greedy ##):
FILENAME="archive.tar.gz"
echo "${FILENAME#*.}"    # Output: tar.gz
echo "${FILENAME##*.}"   # Output: gz (file extension)

# 5. Strip string suffixes (non-greedy % and greedy %%):
FILEPATH="/var/log/nginx/access.log"
echo "${FILEPATH%.log}"  # Output: /var/log/nginx/access
echo "${FILEPATH%/*}"    # Output: /var/log/nginx (dirname)

# 6. Search and Replace:
URL="http://example.com:80"
echo "${URL//http/https}"  # Output: https://example.com:80

# 7. String length:
NAME="Ahmad"
echo "${#NAME}"  # Output: 5
```

---

## 🔀 3. I/O Redirection, Pipes & File Descriptors

### Managing stdout (1) and stderr (2)

| Pattern | Meaning | Example |
| :--- | :--- | :--- |
| `> file` | Overwrite standard output (stdout) | `echo "hi" > log.txt` |
| `>> file` | Append standard output | `echo "hi" >> log.txt` |
| `2> file` | Redirect standard error (stderr) only | `ls /fake 2> errors.log` |
| `&> file` | Redirect **both** stdout and stderr to a file | `./script.sh &> all.log` |
| `2>&1` | Redirect stderr into stdout (POSIX compliant) | `./script.sh > all.log 2>&1` |
| `> /dev/null 2>&1` | Silently discard all outputs | `command > /dev/null 2>&1` |

---

### Dual Logging with `tee`

Print output to the terminal while simultaneously saving it to a log file:

```bash
# Overwrite file and print to screen:
./deploy.sh | tee deploy.log

# Append to file and print to screen:
./deploy.sh | tee -a deploy.log

# Combine stdout + stderr, display both, and append to file:
./deploy.sh 2>&1 | tee -a deploy.log
```

---

### Heredocs for Template Generation

Generate config files or multi-line strings cleanly:

```bash
cat << 'EOF' > /etc/nginx/conf.d/app.conf
server {
    listen 80;
    server_name example.com;
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
}
EOF
```

*(Quoting `'EOF'` prevents variable interpolation. Using unquoted `EOF` allows variables like `${DOMAIN}` inside the template).*

---

## 🔀 4. Conditionals & Control Flow

### The Modern Test Operator `[[ ... ]]`

Always prefer `[[ ... ]]` over legacy `[ ... ]`. It supports regex matching, pattern globbing, and does not break on unquoted whitespace.

```bash
# String comparison:
if [[ "$ENV" == "production" ]]; then
    echo "Running in production"
fi

# Regex matching with =~ :
if [[ "$VERSION" =~ ^v[0-9]+\.[0-9]+ ]]; then
    echo "Valid semantic version format"
fi

# Arithmetic comparison with (( ... )):
if (( COUNT > 10 )); then
    echo "Count exceeded threshold"
fi
```

---

### File & String Test Flags

| Flag | True Condition |
| :--- | :--- |
| `-z "$str"` | String is **empty** (zero length) |
| `-n "$str"` | String is **not empty** |
| `-f "$path"` | Path exists and is a **regular file** |
| `-d "$path"` | Path exists and is a **directory** |
| `-s "$path"` | File exists and has a size **greater than zero** |
| `-e "$path"` | Path exists (file, folder, socket, device) |
| `-x "$path"` | File exists and has **execute permissions** |
| `-r "$path"` | File exists and has **read permissions** |

---

### Loops & Safe Line-by-Line File Processing

Never use `for line in $(cat file.txt)`. It breaks on words and spaces. **Always use a `while` read loop:**

```bash
# Process a file line by line safely:
while IFS= read -r line || [[ -n "$line" ]]; do
    echo "Processing line: $line"
done < "servers.txt"

# Iterate over arrays:
servers=("web01" "web02" "db01")
for server in "${servers[@]}"; do
    echo "Checking $server..."
done
```

---

## 🧩 5. Functions & Exit Codes

### Function Architecture & Arguments

```bash
log_message() {
    local level="$1"
    local message="$2"
    local timestamp
    timestamp="$(date '+%Y-%m-%d %H:%M:%S')"

    echo "[$timestamp] [$level] $message"
}

# Invocation:
log_message "INFO" "Application started successfully"
```

* `$0`: Script name.
* `$1` .. `$9`: Positional arguments.
* `${10}` .. `${N}`: Arguments beyond 9 (curly braces mandatory).
* `$#`: Total count of passed arguments.
* `$@`: Array of all arguments (always quote as `"$@"` to preserve spaces).

---

### Capturing Output vs. Exit Codes

Functions return an **exit code (0–255)** via `return`, while results/data are emitted to **stdout** via `echo`:

```bash
is_port_in_use() {
    local port="$1"
    if ss -tulpn | grep -q ":$port "; then
        return 0  # Port is in use (Success in shell)
    else
        return 1  # Port is free (Failure in shell)
    fi
}

if is_port_in_use 80; then
    echo "Port 80 is occupied!"
fi
```

---

## 🎛️ 6. Command-Line Argument Parsing (`getopts`)

Use the native POSIX `getopts` built-in to parse single-letter command line options:

```bash
#!/usr/bin/env bash
set -euo pipefail

usage() {
    cat << EOF
Usage: $(basename "$0") [-h] [-v] -e ENVIRONMENT -f FILE

Options:
  -e  Target environment (staging | production)
  -f  Path to configuration file
  -v  Enable verbose output
  -h  Display this help message
EOF
    exit 1
}

# Initialize variables
VERBOSE=false
ENV=""
FILE=""

# Parse flags (trailing colon indicates an option takes an argument):
while getopts ":e:f:vh" opt; do
    case "${opt}" in
        e) ENV="${OPTARG}" ;;
        f) FILE="${OPTARG}" ;;
        v) VERBOSE=true ;;
        h) usage ;;
        :) echo "Error: Option -${OPTARG} requires an argument." >&2; usage ;;
        \?) echo "Error: Invalid option -${OPTARG}." >&2; usage ;;
    esac
done
shift $((OPTIND -1))

# Validation:
if [[ -z "$ENV" || -z "$FILE" ]]; then
    echo "Error: Missing mandatory arguments -e or -f." >&2
    usage
fi

echo "Deploying $FILE to $ENV (Verbose: $VERBOSE)"
```

---

## 🛡️ 7. Defensive Scripting: Traps & Locks

### Automatic Cleanup via `trap`

Scripts often generate temporary directories that linger when interrupted by `Ctrl+C` or unexpected crashes. A `trap` guarantees cleanup upon exit:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Create temporary directory securely
TMP_DIR="$(mktemp -d -t myapp-XXXXXX)"

cleanup() {
    echo "Cleaning up temporary files in $TMP_DIR..."
    rm -rf "$TMP_DIR"
}

# Register cleanup function to run on EXIT, SIGINT, or SIGTERM
trap cleanup EXIT INT TERM

# Script payload continues safely:
echo "Doing heavy work in $TMP_DIR..."
touch "$TMP_DIR/payload.bin"
```

---

### Preventing Concurrent Runs with `flock`

Ensure only **one instance** of a backup or cron script runs at any time:

```bash
#!/usr/bin/env bash
set -euo pipefail

LOCK_FILE="/var/run/my-backup.lock"

# Open lock file on file descriptor 200
exec 200>"$LOCK_FILE"

# Try to acquire exclusive, non-blocking lock
if ! flock -n 200; then
    echo "Error: Another instance of this script is already running!" >&2
    exit 1
fi

echo "Lock acquired! Executing critical batch job..."
sleep 10
```

---

## ⏰ 8. Automated Scheduling with Cron

Cron executes scheduled background tasks based on predefined timetables.

### Crontab Syntax & Cheat Sheet

Run `crontab -e` to edit your user's crontab:

```text
┌───────────── Minute (0 - 59)
│ ┌─────────── Hour (0 - 23)
│ │ ┌───────── Day of month (1 - 31)
│ │ │ ┌─────── Month (1 - 12)
│ │ │ │ ┌───── Day of week (0 - 6) (Sunday is 0 or 7)
│ │ │ │ │
* * * * * command_to_execute
```

| Schedule | Syntax | Meaning |
| :--- | :--- | :--- |
| Every 5 minutes | `*/5 * * * *` | Runs every 5th minute |
| Every hour at :30 | `30 * * * *` | e.g. 01:30, 02:30, 03:30 |
| Every night at 2:00 AM | `0 2 * * *` | Typical nightly backup |
| Every Monday at 8:00 AM | `0 8 * * 1` | Weekly report generation |
| Twice a day (Noon & Midnight) | `0 0,12 * * *` | Runs at 00:00 and 12:00 |
| On system boot | `@reboot` | Runs once upon server startup |

---

### Common Cron Gotchas & Mitigations

1. **Restricted `$PATH`**: Cron does **not** load your `~/.bashrc` or full `$PATH`.
   * *Fix*: Define `PATH` explicitly at the top of your crontab, or use absolute paths (`/usr/local/bin/python3`).
2. **Missing Working Directory**:
   * *Fix*: `cd /var/www/app && ./run.sh`.
3. **Lost Error Logs**: By default, Cron tries to email errors locally (which usually fails silently).
   * *Fix*: Always redirect stdout and stderr to a log file:

     ```text
     0 3 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
     ```

---

## 📜 9. Production Script Template

A complete, production-ready script template featuring logging, error traps, arguments, and strict execution:

```bash
#!/usr/bin/env bash
# ==============================================================================
# Script Name: backup_and_sync.sh
# Description: Automated database backup and S3 sync with rotation.
# ==============================================================================
set -euo pipefail
IFS=$'\n\t'

# ------------------------------------------------------------------------------
# Constants & Configuration
# ------------------------------------------------------------------------------
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly TIMESTAMP="$(date +'%Y%m%d_%H%M%S')"
readonly BACKUP_DIR="/var/backups/db"
readonly RETENTION_DAYS=7

# ------------------------------------------------------------------------------
# Logging & Cleanup
# ------------------------------------------------------------------------------
log() {
    local level="$1"
    shift
    printf "[%s] [%s] %s\n" "$(date +'%Y-%m-%d %H:%M:%S')" "$level" "$*"
}

TMP_DIR="$(mktemp -d -t backup-XXXXXX)"
cleanup() {
    log "INFO" "Cleaning up temporary directory: $TMP_DIR"
    rm -rf "$TMP_DIR"
}
trap cleanup EXIT INT TERM

# ------------------------------------------------------------------------------
# Main Execution Logic
# ------------------------------------------------------------------------------
main() {
    log "INFO" "Starting backup routine..."
    mkdir -p "$BACKUP_DIR"

    local archive_name="db_backup_${TIMESTAMP}.tar.gz"
    local target_path="${TMP_DIR}/${archive_name}"

    log "INFO" "Creating database archive: $target_path"
    # Example command (simulated payload):
    touch "$target_path"

    log "INFO" "Moving archive to persistent storage: $BACKUP_DIR"
    mv "$target_path" "${BACKUP_DIR}/${archive_name}"

    log "INFO" "Purging backups older than $RETENTION_DAYS days..."
    find "$BACKUP_DIR" -name "db_backup_*.tar.gz" -type f -mtime +"$RETENTION_DAYS" -delete

    log "INFO" "Backup completed successfully!"
}

main "$@"
```

---

## 🎯 10. Quick Reference Cheat Sheet

| Task | Syntax |
| :--- | :--- |
| **Strict Mode Boilerplate** | `set -euo pipefail` |
| **Safe Directory Resolution** | `SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"` |
| **Fallback Variable Value** | `PORT="${PORT:-8080}"` |
| **Fail if Variable Unset** | `: "${REQUIRED_VAR:?Variable is required!}"` |
| **Redirect stdout & stderr** | `command &> log.txt` or `command > log.txt 2>&1` |
| **Read File Line by Line** | `while IFS= read -r line; do ... done < file.txt` |
| **String is not empty** | `[[ -n "$var" ]]` |
| **File exists & not empty** | `[[ -s "/path/file" ]]` |
| **Clean Temp Files on Exit** | `trap 'rm -rf "$TMP_DIR"' EXIT INT TERM` |
| **Run Daily at 3:00 AM** | `0 3 * * * /opt/script.sh >> /var/log/cron.log 2>&1` |
