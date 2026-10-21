# CS 3307 — Operating Systems 2
## Module 6: Command Shells and Control

---

> **Course Progress:** Module 6 of 8 — CS 3307 (OS 2)

---

## 1. Beyond Basic Scripting

CS 2301 Module 5 introduced shell scripting — variables, loops, conditionals, and simple automation. This module goes deeper into the shell as a programming environment: process control, signals, pipes, advanced scripting patterns, and writing scripts that are robust enough for production use.

---

## 2. The Shell's Role

The shell is both a **command interpreter** (runs commands interactively) and a **scripting language** (automates tasks). When you type a command, the shell:

1. Parses the command line
2. Expands variables, wildcards, and substitutions
3. Forks a child process
4. Executes the command in the child via `exec()`
5. Waits for the child to finish (or runs it in the background)
6. Returns the exit status to the script or prompt

```bash
ls -l /home     # shell forks, execs ls, waits, prints result
ls -l /home &   # shell forks, execs ls, does NOT wait (background)
```

---

## 3. Exit Codes and `$?`

Every command exits with a **status code** — an integer between 0 and 255:

- `0` = success
- Non-zero = failure (specific meaning varies by command)

```bash
ls /tmp
echo $?          # prints 0 (success)

ls /nonexistent
echo $?          # prints 2 (no such file)

grep "root" /etc/passwd
echo $?          # 0 if found, 1 if not found, 2 if error
```

Exit codes are the foundation of shell control flow — `if`, `&&`, and `||` all work based on exit codes.

```bash
# Run command2 only if command1 succeeds (exit code 0)
command1 && command2

# Run command2 only if command1 fails (non-zero exit)
command1 || command2

# Semicolon -- always run both regardless of exit code
command1 ; command2
```

### Setting Exit Codes in Scripts

```bash
#!/bin/bash

if [ ! -f "$1" ]; then
    echo "Error: file not found: $1" >&2   # write to stderr
    exit 1                                  # non-zero = failure
fi

# process the file...
exit 0   # success
```

---

## 4. Special Shell Variables

| Variable | Meaning |
|----------|---------|
| `$0` | Name of the script |
| `$1` .. `$9` | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All arguments as separate words |
| `$*` | All arguments as one string |
| `$$` | PID of the current shell |
| `$!` | PID of the last background process |
| `$?` | Exit code of the last command |
| `$-` | Current shell options |

```bash
#!/bin/bash
echo "Script name:  $0"
echo "First arg:    $1"
echo "All args:     $@"
echo "Arg count:    $#"
echo "My PID:       $$"
```

---

## 5. Advanced Control Flow

### 5.1 `case` Statement

Cleaner than a long chain of `if/elif` for matching a value against patterns:

```bash
#!/bin/bash
day=$(date +%A)   # Monday, Tuesday, etc.

case "$day" in
    Monday|Tuesday|Wednesday|Thursday|Friday)
        echo "Weekday -- time to work"
        ;;
    Saturday|Sunday)
        echo "Weekend -- rest up"
        ;;
    *)
        echo "Unknown day: $day"
        ;;
esac
```

### 5.2 `select` — Interactive Menus

```bash
#!/bin/bash
PS3="Choose an option: "
options=("Start server" "Stop server" "Check status" "Quit")

select opt in "${options[@]}"; do
    case $opt in
        "Start server")  echo "Starting..."; break ;;
        "Stop server")   echo "Stopping..."; break ;;
        "Check status")  echo "Running"; break ;;
        "Quit")          break ;;
        *)               echo "Invalid option" ;;
    esac
done
```

### 5.3 `while read` — Process Input Line by Line

```bash
#!/bin/bash
# Read a file line by line safely
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt
```

- `IFS=` prevents stripping leading/trailing whitespace
- `-r` prevents backslash interpretation
- This is the correct pattern for line-by-line file processing in bash

---

## 6. Functions in Depth

```bash
#!/bin/bash

# Function with return value via exit code
file_exists() {
    [ -f "$1" ]    # returns 0 (true) if file exists, 1 if not
}

# Function returning a value via echo
get_filesize() {
    stat -c%s "$1"   # prints file size in bytes
}

# Usage
if file_exists "/etc/passwd"; then
    echo "File exists"
    size=$(get_filesize "/etc/passwd")
    echo "Size: $size bytes"
fi
```

### Local Variables

```bash
calculate() {
    local result=$(( $1 + $2 ))   # local prevents polluting global scope
    echo "$result"
}

sum=$(calculate 10 20)
echo "Sum: $sum"
```

Always use `local` inside functions — without it, variables are global and can cause subtle bugs in larger scripts.

---

## 7. Pipes and Process Substitution

### 7.1 Pipes

A **pipe** connects the stdout of one command to the stdin of the next — all processes in a pipeline run simultaneously.

```bash
# Count lines containing "ERROR" in a log file
grep "ERROR" server.log | wc -l

# Sort, deduplicate, and count unique IP addresses
cat access.log | cut -d' ' -f1 | sort | uniq -c | sort -rn | head -10

# Show top 5 largest files in current directory
du -sh * | sort -rh | head -5
```

The OS creates a **pipe buffer** between each pair of processes. When the buffer fills, the writer blocks; when it empties, the reader blocks. The kernel manages this automatically.

### 7.2 Named Pipes (FIFOs)

A **named pipe** is a pipe that has a name in the file system — unrelated processes can communicate through it.

```bash
# Terminal 1: create and read from a named pipe
mkfifo /tmp/mypipe
cat /tmp/mypipe      # blocks, waiting for input

# Terminal 2: write to it
echo "Hello from another process" > /tmp/mypipe
```

### 7.3 Process Substitution

Treats the output of a command as a file:

```bash
# Diff the output of two commands without creating temp files
diff <(ls dir1/) <(ls dir2/)

# Read from multiple sources
while IFS= read -r line; do
    echo "Got: $line"
done < <(find /var/log -name "*.log" -newer /tmp/marker)
```

---

## 8. Signals

**Signals** are software interrupts sent to processes. The OS delivers them asynchronously — a process can receive a signal at any point in its execution.

### Common Signals

| Signal | Number | Default Action | Meaning |
|--------|--------|----------------|---------|
| `SIGHUP`  | 1  | Terminate | Hangup (terminal closed) |
| `SIGINT`  | 2  | Terminate | Interrupt (Ctrl+C) |
| `SIGQUIT` | 3  | Core dump  | Quit (Ctrl+\\) |
| `SIGKILL` | 9  | Terminate  | Kill (cannot be caught or ignored) |
| `SIGTERM` | 15 | Terminate  | Polite termination request |
| `SIGSTOP` | 19 | Stop       | Pause process (cannot be caught) |
| `SIGCONT` | 18 | Continue   | Resume a stopped process |
| `SIGUSR1` | 10 | Terminate  | User-defined signal 1 |
| `SIGUSR2` | 12 | Terminate  | User-defined signal 2 |

### Sending Signals

```bash
kill -15 1234     # send SIGTERM to PID 1234 (polite stop)
kill -9  1234     # send SIGKILL  (force kill -- cannot be ignored)
kill -1  1234     # send SIGHUP   (reload config for many daemons)

pkill nginx       # send SIGTERM to all processes named "nginx"
killall python3   # same but exact name match
```

### Trapping Signals in Scripts

```bash
#!/bin/bash

# Cleanup function
cleanup() {
    echo "Caught signal -- cleaning up temp files..."
    rm -f /tmp/myapp_$$.*
    exit 1
}

# Register signal handlers
trap cleanup SIGINT SIGTERM

# Create a temp file
tmpfile="/tmp/myapp_$$.data"
echo "working" > "$tmpfile"

echo "Running... press Ctrl+C to stop"
while true; do
    sleep 1
    echo "still running..."
done
```

`trap` registers a function (or command) to run when a signal is received. This is essential for scripts that create temp files, hold locks, or leave the system in a partially changed state — the cleanup must run even if the user hits Ctrl+C.

---

## 9. Process Control

### Running Jobs in the Background

```bash
long_task.sh &          # run in background; shell prints PID
echo $!                 # PID of the last background job

# Bring to foreground
fg %1                   # bring job 1 to foreground

# Send to background (if already running)
Ctrl+Z                  # pause (SIGSTOP) current foreground job
bg %1                   # resume it in the background

jobs                    # list all background jobs
```

### `wait` — Synchronize Background Jobs

```bash
#!/bin/bash

# Run three tasks in parallel
process_chunk.sh chunk1 &
process_chunk.sh chunk2 &
process_chunk.sh chunk3 &

# Wait for ALL background jobs to finish
wait
echo "All chunks processed"
```

### `wait` with Specific PID

```bash
process_chunk.sh chunk1 &
pid1=$!

process_chunk.sh chunk2 &
pid2=$!

wait $pid1
echo "chunk1 done (exit code: $?)"

wait $pid2
echo "chunk2 done (exit code: $?)"
```

---

## 10. Here Documents and Here Strings

### Here Document

Embed multi-line text directly in a script without a separate file:

```bash
#!/bin/bash

cat << EOF
Dear $USER,

Your backup completed at $(date).
Files backed up: $(ls /backup | wc -l)

Regards,
The Backup System
EOF
```

Useful for generating config files, SQL queries, emails, or any multi-line text with variable expansion.

```bash
# Send a query to a database
mysql -u root -p mydb << EOF
SELECT id, name, email
FROM users
WHERE created_at > NOW() - INTERVAL 7 DAY;
EOF
```

### Here String

Feed a single string to a command's stdin:

```bash
grep "error" <<< "this line has an error in it"
read first rest <<< "hello world foo"
echo $first   # hello
```

---

## 11. Robust Script Patterns

### 11.1 `set` Options for Safer Scripts

```bash
#!/bin/bash
set -e          # exit immediately on any error
set -u          # treat unset variables as errors
set -o pipefail # pipeline fails if any command in it fails
set -x          # print each command before executing (debug mode)

# Common combination at the top of production scripts:
set -euo pipefail
```

Without `set -e`, a script will happily continue after a failed command — often causing cascading failures that are hard to debug.

### 11.2 Locking — Prevent Duplicate Script Runs

```bash
#!/bin/bash
LOCKFILE="/tmp/myapp.lock"

# Acquire lock or exit if already running
exec 9>"$LOCKFILE"
flock -n 9 || { echo "Already running"; exit 1; }

echo "Running with PID $$"
# ... do work ...
# Lock is automatically released when the script exits
```

### 11.3 Temporary Files Done Right

```bash
#!/bin/bash

# mktemp creates a uniquely named temp file
tmpfile=$(mktemp /tmp/myapp.XXXXXX)

# Always clean up, even on error
trap "rm -f $tmpfile" EXIT   # EXIT trap runs on any exit

echo "data" > "$tmpfile"
process "$tmpfile"
# tmpfile automatically deleted when script exits
```

`trap ... EXIT` is the safest cleanup pattern — it runs whether the script exits normally, on error, or due to a signal.

---

## 12. Real-World Example: A Production Deployment Script

```bash
#!/bin/bash
set -euo pipefail

APP_DIR="/var/www/myapp"
BACKUP_DIR="/var/backups/myapp"
REPO_URL="https://github.com/myorg/myapp"
LOG="/var/log/deploy.log"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG"; }
die() { log "ERROR: $*"; exit 1; }

cleanup() { log "Deploy script exiting (PID $$)"; }
trap cleanup EXIT

log "Starting deployment"

# Validate environment
[ -d "$APP_DIR" ]   || die "App directory not found: $APP_DIR"
command -v git      || die "git not installed"
command -v systemctl|| die "systemctl not available"

# Backup current version
log "Backing up current version..."
timestamp=$(date +%Y%m%d_%H%M%S)
cp -r "$APP_DIR" "$BACKUP_DIR/backup_$timestamp"

# Pull latest code
log "Pulling latest code..."
cd "$APP_DIR"
git pull origin main

# Restart service
log "Restarting application..."
systemctl restart myapp || die "Failed to restart service"

# Verify it started
sleep 2
systemctl is-active --quiet myapp || die "Service failed to start"

log "Deployment complete"
```

This script demonstrates: `set -euo pipefail`, logging with timestamps, a `die()` helper, `trap` for cleanup, input validation with `die`, and structured steps with clear logging.

---

## 13. Summary

| Concept | Key Takeaway |
|---------|-------------|
| Exit codes | 0 = success, non-zero = failure; foundation of shell control flow |
| `$?`, `$#`, `$@`, `$$` | Essential special variables |
| `case` statement | Clean multi-branch matching |
| `while IFS= read -r` | Correct pattern for line-by-line file processing |
| Pipes | Connect stdout of one process to stdin of next; run simultaneously |
| Named pipes (FIFO) | Pipes with a filesystem name for unrelated processes |
| Signals | Asynchronous software interrupts; `trap` handles them in scripts |
| `SIGKILL` vs `SIGTERM` | SIGTERM can be caught; SIGKILL cannot |
| `trap` | Register cleanup functions for signals and EXIT |
| Background jobs | `&`, `wait`, `fg`, `bg`, `jobs` |
| Here documents | Embed multi-line text with variable expansion in scripts |
| `set -euo pipefail` | Makes scripts fail fast and safely |
| `flock` | Prevent duplicate script instances from running |
| `mktemp` + `trap EXIT` | Safe temp file pattern |

---

## Module 6 Quiz

---

**Question 1**
What does `set -euo pipefail` do and why should production scripts use it?

> **Answer:**
> - `set -e` — exit immediately if any command returns a non-zero exit code (instead of silently continuing)
> - `set -u` — treat any reference to an unset variable as an error (catches typos like `$DIIR` instead of `$DIR`)
> - `set -o pipefail` — a pipeline fails if any command in it fails, not just the last one (without this, `false | true` succeeds)
>
> Production scripts use it because the default shell behavior — silently ignoring errors and continuing — can cause catastrophic cascading failures. With these options, a script fails loudly and immediately at the first sign of trouble.

---

**Question 2**
What is the difference between `SIGTERM` and `SIGKILL`? When would you use each?

> **Answer:** `SIGTERM` (signal 15) is a polite termination request — the process receives it and can catch it with a signal handler to perform cleanup (close files, release locks, finish current work) before exiting. `SIGKILL` (signal 9) is an unconditional kill — it is sent directly by the kernel and cannot be caught, blocked, or ignored by the process. Use `SIGTERM` first and give the process a few seconds to shut down cleanly. Use `SIGKILL` only if the process does not respond to SIGTERM — it guarantees termination but risks leaving resources in an inconsistent state.

---

**Question 3**
A script creates a temporary file at the start and must delete it when finished, even if the script crashes or the user presses Ctrl+C. Write the correct bash pattern for this.

> **Answer:**
> ```bash
> #!/bin/bash
> tmpfile=$(mktemp /tmp/myapp.XXXXXX)
> trap "rm -f $tmpfile" EXIT
>
> # ... rest of script ...
> # tmpfile is automatically deleted on any exit:
> # normal finish, error (set -e), SIGINT (Ctrl+C), SIGTERM
> ```
> The `EXIT` trap fires whenever the shell exits for any reason. `mktemp` ensures a unique filename that won't collide with other running instances.

---

**Question 4**
Explain what this pipeline does and how the OS executes it:

```bash
grep "ERROR" app.log | sort | uniq -c | sort -rn | head -5
```

> **Answer:** The pipeline finds all lines containing "ERROR" in `app.log`, sorts them alphabetically (so identical lines are adjacent), counts consecutive duplicate lines (`uniq -c`), sorts by count in reverse numeric order (most frequent first), and shows the top 5. It prints the 5 most common error messages and how many times each occurred.
>
> The OS creates 4 pipes and forks 5 processes simultaneously. Each process reads from its stdin pipe and writes to its stdout pipe. The kernel manages the pipe buffers — when a buffer fills, the writer blocks; when it empties, the reader blocks. All 5 processes run in parallel as a pipeline, which is more efficient than running them sequentially.

---

**Question 5**
Write a bash script that accepts a directory path as an argument, verifies it exists, then prints the name and size of every `.log` file in that directory, sorted by size largest-first.

> **Answer:**
> ```bash
> #!/bin/bash
> set -euo pipefail
>
> if [ $# -ne 1 ]; then
>     echo "Usage: $0 <directory>" >&2
>     exit 1
> fi
>
> dir="$1"
>
> if [ ! -d "$dir" ]; then
>     echo "Error: not a directory: $dir" >&2
>     exit 1
> fi
>
> echo "Log files in $dir (largest first):"
> find "$dir" -maxdepth 1 -name "*.log" -type f \
>     -exec stat -c "%s %n" {} \; \
>     | sort -rn \
>     | while IFS= read -r line; do
>         size=$(echo "$line" | cut -d' ' -f1)
>         name=$(echo "$line" | cut -d' ' -f2-)
>         printf "%-12d %s\n" "$size" "$(basename $name)"
>     done
> ```

---

## What's Next

**Module 7: Operating System Utilities**

You will learn the essential Unix/Linux utilities that every OS professional uses daily — text processing tools (`awk`, `sed`, `grep`), system monitoring, process management, network tools, and how to chain them together effectively.

---

*CS 3307 Operating Systems 2 — Module 6 of 8*
