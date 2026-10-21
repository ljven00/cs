# CS 3307 — Operating Systems 2
## Module 7: Operating System Utilities

---

> **Course Progress:** Module 7 of 8 — CS 3307 (OS 2)

---

## 1. Why OS Utilities Matter

An OS is only as useful as the tools it provides. Unix/Linux ships with a powerful set of utilities — small programs that each do one thing well and can be combined via pipes into sophisticated workflows. Mastering these tools is what separates a competent developer from a productive one.

This module covers the essential utilities across five categories: text processing, file management, system monitoring, process management, and networking.

---

## 2. Text Processing Utilities

### 2.1 `grep` — Search Text

```bash
# Basic search
grep "ERROR" app.log

# Case-insensitive
grep -i "error" app.log

# Show line numbers
grep -n "ERROR" app.log

# Invert match (lines NOT containing pattern)
grep -v "DEBUG" app.log

# Recursive search through directories
grep -r "TODO" ./src/

# Count matching lines
grep -c "ERROR" app.log

# Extended regex (alternation, +, ?)
grep -E "ERROR|WARN|FATAL" app.log

# Show N lines of context around each match
grep -A 2 -B 2 "ERROR" app.log    # 2 lines after and before
```

### 2.2 `sed` — Stream Editor

`sed` processes text line by line, applying transformations.

```bash
# Substitute first occurrence per line
sed 's/old/new/' file.txt

# Substitute ALL occurrences per line (global flag)
sed 's/old/new/g' file.txt

# In-place edit (modifies the file directly)
sed -i 's/localhost/production.db/g' config.txt

# Delete lines matching a pattern
sed '/^#/d' config.txt          # delete comment lines

# Print only lines 10-20
sed -n '10,20p' file.txt

# Insert a line before line 5
sed '5i\New line inserted here' file.txt

# Chain multiple commands
sed -e 's/foo/bar/g' -e '/^$/d' file.txt   # replace and remove blank lines
```

### 2.3 `awk` — Pattern Scanning and Processing

`awk` processes text field by field. It is a complete mini-language.

```bash
# Print specific columns (fields) from whitespace-separated data
awk '{print $1, $3}' data.txt          # columns 1 and 3

# Use a custom delimiter
awk -F',' '{print $2}' data.csv        # second column of CSV

# Filter rows by condition
awk '$3 > 90 {print $0}' scores.txt    # lines where column 3 > 90

# Compute a sum
awk -F',' '{sum += $3} END {print "Total:", sum}' scores.csv

# Print line count (like wc -l but more flexible)
awk 'END {print NR}' file.txt

# Print lines between two patterns
awk '/START/,/END/ {print}' file.txt
```

### `awk` Built-In Variables

| Variable | Meaning |
|----------|---------|
| `NR` | Current line number |
| `NF` | Number of fields in current line |
| `$0` | Entire current line |
| `$1..$NF` | Individual fields |
| `FS` | Field separator (default: whitespace) |
| `OFS` | Output field separator |

### 2.4 `cut` — Extract Columns

```bash
# Extract characters 1-10
cut -c1-10 file.txt

# Extract field 2 from CSV
cut -d',' -f2 data.csv

# Extract fields 1 and 3
cut -d':' -f1,3 /etc/passwd
```

### 2.5 `sort` and `uniq`

```bash
# Sort alphabetically
sort names.txt

# Sort numerically
sort -n scores.txt

# Sort by column 3, numerically, reverse (largest first)
sort -t',' -k3 -rn scores.csv

# Remove duplicate lines (input must be sorted)
sort names.txt | uniq

# Count occurrences
sort names.txt | uniq -c

# Show only duplicates
sort names.txt | uniq -d
```

### 2.6 `tr` — Translate Characters

```bash
# Convert lowercase to uppercase
echo "hello world" | tr 'a-z' 'A-Z'

# Delete characters
echo "hello 123 world" | tr -d '0-9'    # removes digits

# Squeeze repeated characters
echo "hello     world" | tr -s ' '      # multiple spaces -> one
```

### 2.7 `wc` — Word/Line/Byte Count

```bash
wc -l file.txt      # line count
wc -w file.txt      # word count
wc -c file.txt      # byte count
wc file.txt         # all three
```

---

## 3. File Management Utilities

### 3.1 `find` — Search the Filesystem

```bash
# Find by name
find /home -name "*.log"

# Find files modified in the last 24 hours
find /var/log -mtime -1 -type f

# Find files larger than 100MB
find / -size +100M -type f

# Find and delete old log files
find /var/log/myapp -name "*.log" -mtime +30 -delete

# Find and execute a command on each result
find . -name "*.txt" -exec wc -l {} \;

# Find by permission
find /etc -perm 644 -type f

# Find empty files or directories
find . -empty
```

### 3.2 `xargs` — Build Commands from Input

`xargs` reads lines from stdin and passes them as arguments to a command — essential when `find` results need to be processed.

```bash
# Delete all .tmp files found by find
find /tmp -name "*.tmp" | xargs rm -f

# Count lines in all Python files
find . -name "*.py" | xargs wc -l

# Process in parallel (4 at a time)
find . -name "*.log" | xargs -P 4 gzip

# Handle filenames with spaces safely
find . -name "*.txt" -print0 | xargs -0 grep "TODO"
```

### 3.3 `tar` — Archive and Compress

```bash
# Create a compressed archive
tar -czf backup.tar.gz /home/alice/

# Extract
tar -xzf backup.tar.gz

# List contents without extracting
tar -tzf backup.tar.gz

# Create archive with bzip2 (better compression, slower)
tar -cjf backup.tar.bz2 /home/alice/

# Extract to specific directory
tar -xzf backup.tar.gz -C /restore/
```

| Flag | Meaning |
|------|---------|
| `c` | Create archive |
| `x` | Extract archive |
| `t` | List contents |
| `z` | Use gzip compression |
| `j` | Use bzip2 compression |
| `J` | Use xz compression |
| `f` | Specify filename |
| `v` | Verbose (show files) |

### 3.4 `rsync` — Efficient File Synchronization

```bash
# Sync a local directory to another location
rsync -av /home/alice/ /backup/alice/

# Sync to a remote server over SSH
rsync -avz /home/alice/ user@server:/backup/alice/

# Dry run (show what would be transferred, don't do it)
rsync -avn /home/alice/ /backup/alice/

# Delete files in destination that no longer exist in source
rsync -av --delete /home/alice/ /backup/alice/
```

`rsync` only transfers files that have changed — far more efficient than `cp` for incremental backups.

---

## 4. System Monitoring Utilities

### 4.1 `top` / `htop` — Live Process Monitor

```bash
top       # built-in, always available
htop      # improved version, color, mouse support (may need install)
```

Key columns in `top`:

| Column | Meaning |
|--------|---------|
| PID | Process ID |
| USER | Owner |
| %CPU | CPU usage percentage |
| %MEM | Memory usage percentage |
| VSZ | Virtual memory size |
| RSS | Resident set size (actual RAM used) |
| COMMAND | Process name |

Interactive keys in `top`:
- `k` — kill a process (enter PID)
- `r` — renice (change priority)
- `M` — sort by memory
- `P` — sort by CPU
- `q` — quit

### 4.2 `ps` — Process Snapshot

```bash
# All processes, full format
ps aux

# Process tree (show parent-child relationships)
ps auxf

# Find a specific process
ps aux | grep nginx

# Show processes by user
ps -u alice

# Custom output columns
ps -eo pid,ppid,user,%cpu,%mem,command --sort=-%cpu | head -10
```

### 4.3 `vmstat` — Virtual Memory Statistics

```bash
vmstat 2 5    # print stats every 2 seconds, 5 times
```

```
procs ----memory---- -swap- ---io--- -system- ----cpu----
 r  b  swpd   free   buff  cache  si  so  bi  bo  in  cs us sy id wa
 1  0     0 512000  12000 800000   0   0   5  10 200 400  5  2 92  1
```

Key columns: `r` (processes waiting for CPU), `b` (blocked on I/O), `si`/`so` (swap in/out — non-zero means thrashing), `us`/`sy`/`id` (CPU user/system/idle %).

### 4.4 `iostat` — Disk I/O Statistics

```bash
iostat -x 2    # extended stats every 2 seconds
```

Useful for spotting disk bottlenecks — high `%util` (near 100%) means the disk is saturated.

### 4.5 `df` and `du` — Disk Space

```bash
df -h           # disk space by filesystem (human-readable)
df -i           # inode usage (can run out of inodes before disk space!)

du -sh /home/*  # size of each user's home directory
du -sh * | sort -rh | head -10   # top 10 largest items in current dir
```

### 4.6 `free` — Memory Usage

```bash
free -h         # memory usage in human-readable form
```

```
              total    used    free   shared  buff/cache  available
Mem:           15Gi    4.2Gi   8.1Gi   512Mi     2.9Gi      10.8Gi
Swap:           2Gi      0B    2Gi
```

`available` is more useful than `free` — it includes memory that can be freed from caches.

### 4.7 `lsof` — List Open Files

```bash
lsof                    # all open files (huge list)
lsof -p 1234            # files opened by process 1234
lsof /var/log/app.log   # processes that have this file open
lsof -i :8080           # process listening on port 8080
lsof -u alice           # all files opened by user alice
```

`lsof` is invaluable for debugging — finding what has a file locked, what is listening on a port, or what a process is doing.

---

## 5. Process Management Utilities

### 5.1 `nice` and `renice` — Process Priority

The **nice value** ranges from -20 (highest priority) to 19 (lowest). Default is 0.

```bash
# Start a process with low priority (won't compete with interactive work)
nice -n 19 ./heavy_computation.sh

# Change priority of a running process
renice -n 10 -p 1234        # lower priority of PID 1234
renice -n -5 -p 1234        # raise priority (requires root)
```

### 5.2 `nohup` — Survive Shell Exit

```bash
# Run a command that keeps going after you log out
nohup ./long_task.sh > output.log 2>&1 &
echo $!    # note the PID
```

Without `nohup`, processes receive `SIGHUP` when the terminal closes and typically exit.

### 5.3 `screen` and `tmux` — Terminal Multiplexers

Allow multiple terminal sessions inside one, and keep sessions alive after disconnection.

```bash
# tmux basics
tmux new -s mysession       # create named session
tmux attach -t mysession    # reattach after disconnect
Ctrl+b d                    # detach (session keeps running)
Ctrl+b c                    # new window
Ctrl+b n                    # next window
Ctrl+b %                    # split pane vertically
```

Essential for long-running jobs on remote servers.

### 5.4 `strace` — Trace System Calls

```bash
# Show all system calls made by a program
strace ls /tmp

# Trace a running process
strace -p 1234

# Count system calls
strace -c ls /tmp

# Filter by system call type
strace -e trace=openat,read,write ls /tmp
```

`strace` is a powerful debugging tool — you can see exactly what files a program opens, what network connections it makes, and where it is getting stuck.

---

## 6. Networking Utilities

### 6.1 `curl` — Transfer Data with URLs

```bash
# Download a file
curl -O https://example.com/file.zip

# Make a GET request and show response
curl https://api.example.com/users

# POST JSON data
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"Alice"}' \
     https://api.example.com/users

# Follow redirects, show HTTP status code
curl -L -w "%{http_code}\n" https://example.com

# Download only if newer than local file
curl -z localfile.txt -O https://example.com/data.txt
```

### 6.2 `wget` — Download Files

```bash
# Download a file
wget https://example.com/file.zip

# Download recursively (mirror a website)
wget -r -np https://example.com/docs/

# Continue an interrupted download
wget -c https://example.com/bigfile.iso
```

### 6.3 `ss` / `netstat` — Network Connections

```bash
# Show all listening ports
ss -tlnp

# Show all established connections
ss -tnp

# Show UDP sockets
ss -unp

# (older alternative)
netstat -tlnp
```

### 6.4 `ping`, `traceroute`, `dig`

```bash
# Test connectivity
ping -c 4 google.com

# Trace the route packets take
traceroute google.com

# DNS lookup
dig google.com
dig +short google.com      # just the IP
dig -x 8.8.8.8             # reverse lookup
```

### 6.5 `ssh` — Secure Shell

```bash
# Connect to remote server
ssh user@server.example.com

# Run a single command remotely
ssh user@server "df -h"

# Copy files securely
scp file.txt user@server:/home/user/
scp -r dir/ user@server:/home/user/

# SSH tunneling (forward local port 8080 to remote port 80)
ssh -L 8080:localhost:80 user@server
```

---

## 7. Combining Utilities — Real Workflows

### Find the Top 10 Most Active IP Addresses in a Web Log

```bash
awk '{print $1}' access.log \
    | sort \
    | uniq -c \
    | sort -rn \
    | head -10
```

### Find All Files Modified in the Last Hour and Archive Them

```bash
find /var/data -newer /tmp/marker -type f \
    | xargs tar -czf /backup/recent_$(date +%H%M).tar.gz
```

### Monitor Disk Space and Alert if Over 80%

```bash
#!/bin/bash
df -h | awk 'NR>1 {gsub(/%/,"",$5); if ($5 > 80) print "WARNING: "$6" is "$5"% full"}'
```

### Watch a Log File in Real Time

```bash
tail -f /var/log/app.log | grep --line-buffered "ERROR"
```

### Kill All Processes Matching a Pattern

```bash
ps aux | grep "[p]ython3" | awk '{print $2}' | xargs kill -15
# The [p] trick prevents grep from matching its own process
```

---

## 8. Summary

| Tool | Purpose |
|------|---------|
| `grep` | Search text for patterns |
| `sed` | Stream edit — substitute, delete, transform |
| `awk` | Field-based text processing and computation |
| `cut` | Extract specific columns |
| `sort` / `uniq` | Sort and deduplicate |
| `tr` | Translate or delete characters |
| `wc` | Count lines, words, bytes |
| `find` | Search filesystem by name, size, time, permissions |
| `xargs` | Pass stdin lines as command arguments |
| `tar` | Archive and compress files |
| `rsync` | Efficient incremental file synchronization |
| `top` / `htop` | Live process and resource monitor |
| `ps` | Process snapshot |
| `vmstat` | Virtual memory and CPU stats |
| `df` / `du` | Disk space by filesystem and directory |
| `lsof` | List open files, ports, and locks |
| `nice` / `renice` | Adjust process scheduling priority |
| `strace` | Trace system calls of a process |
| `curl` | HTTP requests and file transfers |
| `ss` | Network socket status |
| `ssh` / `scp` | Remote login and secure file copy |

---

## Module 7 Quiz

---

**Question 1**
What does this pipeline do? Break it down step by step.

```bash
ps aux | awk '{print $1}' | sort | uniq -c | sort -rn | head -5
```

> **Answer:** Step by step:
> 1. `ps aux` — list all running processes with full details
> 2. `awk '{print $1}'` — extract only the first column (the username/owner of each process)
> 3. `sort` — sort the usernames alphabetically so identical names are adjacent
> 4. `uniq -c` — count consecutive duplicate lines, prepending the count to each unique username
> 5. `sort -rn` — sort by count numerically in reverse order (most processes first)
> 6. `head -5` — show only the top 5 results
>
> Overall: it shows the **5 users running the most processes**, ordered from most to fewest.

---

**Question 2**
What is the difference between `grep`, `sed`, and `awk`? When would you reach for each one?

> **Answer:**
> - `grep` — finds and filters lines that match a pattern. Use it when you want to select which lines to keep or count matches. It does not transform data.
> - `sed` — transforms text line by line, primarily for substitution (`s/old/new/g`), deletion, and insertion. Use it when you want to modify text in a predictable, line-oriented way.
> - `awk` — processes text field by field with a full programming model (variables, arithmetic, conditionals, `BEGIN`/`END` blocks). Use it when you need to compute values, reformat columns, filter by numeric conditions, or aggregate data.
>
> Quick rule: filter lines → `grep`; transform lines → `sed`; compute from fields → `awk`.

---

**Question 3**
A server is running slowly. Describe how you would use three different utilities to diagnose whether the bottleneck is CPU, memory, or disk I/O.

> **Answer:**
> - **CPU:** Run `top` or `htop`. Look at the `%id` (idle) column in the CPU summary — if it is near 0%, the CPU is saturated. Check which processes have high `%CPU`. Also check `vmstat` — a high `r` value (processes waiting for CPU) confirms CPU contention.
> - **Memory:** Run `free -h`. If `available` memory is very low and `swap` is being used (non-zero `used` swap), the system is swapping — a memory bottleneck. `vmstat` shows `si`/`so` (swap in/out); non-zero values confirm thrashing.
> - **Disk I/O:** Run `iostat -x 2`. Look for devices with `%util` near 100% — that disk is saturated. Also check `vmstat`'s `wa` column (CPU time waiting for I/O); high values mean processes are blocked on disk.

---

**Question 4**
What does `lsof -i :8080` tell you, and when would you use it?

> **Answer:** It lists all processes that have a network socket open on port 8080 — showing the process name, PID, user, and whether the socket is listening or established. You would use it when a service fails to start because "address already in use" — it tells you exactly which process is already holding that port. It is also useful for security audits (finding unexpected listeners) and debugging connection issues.

---

**Question 5**
Write a one-liner that finds the 3 largest files under `/var/log`, displays their sizes in human-readable format, and shows only the filename (not the full path).

> **Answer:**
> ```bash
> find /var/log -type f -exec du -h {} \; | sort -rh | head -3 | awk '{print $1, $2}' | xargs -I{} bash -c 'echo "$1 $(basename $2)"' -- {}
> ```
>
> Simpler alternative that works well in practice:
> ```bash
> find /var/log -type f -printf "%s %f\n" | sort -rn | head -3 | awk '{printf "%d KB\t%s\n", $1/1024, $2}'
> ```
>
> Or using `du` with `basename`:
> ```bash
> find /var/log -type f | xargs du -h | sort -rh | head -3 | awk '{cmd="basename "$2; cmd | getline base; close(cmd); print $1, base}'
> ```

---

## What's Next

**Module 8: Graphical User Interfaces**

The final module of CS 3307 — you will learn how GUIs are built on top of the OS, the role of display servers (X11, Wayland), window managers, GUI toolkits, and how graphical applications interact with the underlying system.

---

*CS 3307 Operating Systems 2 — Module 7 of 8*
