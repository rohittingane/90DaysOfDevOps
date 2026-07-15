# 🐚 Shell Scripting Cheat Sheet — Day 21 (#90DaysOfDevOps)

Personal quick-reference guide built after learning Shell scripting from basics to real-world projects.

---

## 📌 Quick Reference Table

| Topic | Key Syntax | Example |
|---|---|---|
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Argument | `$1`, `$2` | `./script.sh arg1` |
| If | `if [ condition ]; then` | `if [ -f file ]; then` |
| For loop | `for i in list; do` | `for i in 1 2 3; do` |
| Function | `name() { ... }` | `greet() { echo "Hi"; }` |
| Grep | `grep pattern file` | `grep -i "error" log.txt` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed -i 's/foo/bar/g' config.txt` |

---

## 1️⃣ Basics

### Shebang
First line of every script. Tells the OS which interpreter to use to run the script.
> 💡 **Real-time use:** Every deployment/automation script (Ansible, cron, Jenkins shell steps) starts with this so it runs consistently across different servers, regardless of the default shell.
```bash
#!/bin/bash
echo "Hello, DevOps!"
```
Without it, the script may run with the wrong shell and behave unexpectedly.

### Running a Script
> 💡 **Real-time use:** In CI/CD pipelines you often see `chmod +x deploy.sh && ./deploy.sh` as a build step before execution.
```bash
chmod +x script.sh   # make it executable
./script.sh           # run using the shebang interpreter
bash script.sh        # run explicitly with bash (no +x needed)
```

### Comments
> 💡 **Real-time use:** Documenting *why* a cron job or workaround exists, so the next engineer (or future you) doesn't delete it by mistake.
```bash
# This is a single line comment
echo "Hello"   # this is an inline comment
```

### Variables
> 💡 **Real-time use:** Storing config like environment name, API URLs, or file paths at the top of a script so they can be changed in one place.
```bash
NAME="Shubham"        # no spaces around =
echo $NAME             # unquoted - word splitting can occur
echo "$NAME"           # quoted - preserves spaces, safest to use
echo '$NAME'           # single quotes - literal, no expansion
```

### Reading User Input
> 💡 **Real-time use:** Interactive setup/installer scripts that ask "Which environment do you want to deploy to? (dev/staging/prod)".
```bash
read -p "Enter your name: " NAME
echo "Hello, $NAME"
```

### Command-Line Arguments
> 💡 **Real-time use:** systemd/init scripts and custom CLI tools like `./deploy.sh production` or `./backup.sh --force` rely entirely on this.
```bash
#!/bin/bash
echo "Script name: $0"
echo "First arg: $1"
echo "Total args: $#"
echo "All args: $@"
echo "Exit status of last command: $?"
```

---

## 2️⃣ Operators and Conditionals

### String Comparisons
> 💡 **Real-time use:** Checking which environment a script is running in, e.g. `if [ "$ENV" = "production" ]` before applying stricter safety checks.
```bash
[ "$a" = "$b" ]    # equal
[ "$a" != "$b" ]   # not equal
[ -z "$a" ]        # true if string is empty
[ -n "$a" ]        # true if string is NOT empty
```

### Integer Comparisons
> 💡 **Real-time use:** Disk/CPU/memory monitoring scripts comparing usage percentage against a threshold, e.g. `[ "$DISK" -gt 80 ]`.
```bash
[ "$a" -eq "$b" ]  # equal
[ "$a" -ne "$b" ]  # not equal
[ "$a" -lt "$b" ]  # less than
[ "$a" -gt "$b" ]  # greater than
[ "$a" -le "$b" ]  # less than or equal
[ "$a" -ge "$b" ]  # greater than or equal
```

### File Test Operators
> 💡 **Real-time use:** Deployment scripts checking `[ -f .env ]` before starting an app, or `[ -x script.sh ]` before running it in a pipeline.
```bash
[ -f file ]   # regular file exists
[ -d dir ]    # directory exists
[ -e path ]   # path exists (file or dir)
[ -r file ]   # readable
[ -w file ]   # writable
[ -x file ]   # executable
[ -s file ]   # file exists and is not empty
```

### if / elif / else
> 💡 **Real-time use:** Deployment logic that decides "if config file exists, use it; elif a template exists, generate one; else fail".
```bash
if [ -f "file.txt" ]; then
    echo "File exists"
elif [ -d "file.txt" ]; then
    echo "It's a directory"
else
    echo "Not found"
fi
```

### Logical Operators
> 💡 **Real-time use:** Chaining a health check with an action — `curl -sf http://localhost:8080 || systemctl restart myapp`.
```bash
[ -f file ] && echo "exists"          # AND - run if true
[ -f file ] || echo "not found"       # OR - run if false
! [ -f file ] && echo "missing"       # NOT
```

### Case Statements
> 💡 **Real-time use:** Classic `service` control scripts (`/etc/init.d/*`) that respond to `start`, `stop`, `restart`, `status` arguments.
```bash
case "$1" in
    start) echo "Starting..." ;;
    stop)  echo "Stopping..." ;;
    *)     echo "Usage: $0 {start|stop}" ;;
esac
```

---

## 3️⃣ Loops

### For Loop
> 💡 **Real-time use:** Looping through a list of servers to run the same deployment command via SSH on each one.
```bash
# list-based
for fruit in apple banana mango; do
    echo "$fruit"
done

# C-style
for ((i=0; i<5; i++)); do
    echo "$i"
done
```

### While Loop
> 💡 **Real-time use:** Retry logic — keep checking every few seconds until a newly deployed service becomes healthy.
```bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### Until Loop
> 💡 **Real-time use:** Waiting until a port opens (e.g. database becoming ready) before starting the app that depends on it.
```bash
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### Loop Control
> 💡 **Real-time use:** Skipping already-processed backup files (`continue`) or stopping a scan early once an error threshold is hit (`break`).
```bash
for i in 1 2 3 4 5; do
    [ $i -eq 3 ] && continue   # skip 3
    [ $i -eq 5 ] && break      # stop at 5
    echo "$i"
done
```

### Looping Over Files
> 💡 **Real-time use:** Compressing or archiving all `.log` files in a directory during log rotation.
```bash
for file in *.log; do
    echo "Processing $file"
done
```

### Looping Over Command Output
> 💡 **Real-time use:** Reading a `servers.txt` inventory file line by line to SSH into each server and run a health check.
```bash
cat servers.txt | while read line; do
    echo "Server: $line"
done
```

---

## 4️⃣ Functions

### Defining and Calling
> 💡 **Real-time use:** A reusable `log()` function used everywhere in a script to print consistent timestamped messages instead of repeating `echo` everywhere.
```bash
greet() {
    echo "Hello, $1!"
}
greet "DevOps"
```

### Arguments Inside Functions
> 💡 **Real-time use:** A generic `restart_service()` function that takes the service name as an argument, so it can restart nginx, mysql, or any app the same way.
```bash
add() {
    echo "Sum: $(( $1 + $2 ))"
}
add 5 10
```

### Return Values
> 💡 **Real-time use:** A `check_service_health()` function returning 0/1 (success/failure) that the main script uses to decide whether to send an alert.
```bash
check_even() {
    if (( $1 % 2 == 0 )); then
        return 0   # success/true
    else
        return 1   # failure/false
    fi
}
check_even 4 && echo "Even"

# echo is used to actually return data (not just status)
get_sum() {
    echo $(( $1 + $2 ))
}
result=$(get_sum 3 4)
```

### Local Variables
> 💡 **Real-time use:** Preventing a variable inside a helper function from accidentally overwriting a same-named variable in a large, multi-function deployment script.
```bash
myfunc() {
    local temp="only visible inside function"
    echo "$temp"
}
```

---

## 5️⃣ Text Processing Commands

### grep — search patterns
> 💡 **Real-time use:** Searching application logs for `"error"` or `"exception"` during an incident to quickly find what went wrong.
```bash
grep "error" file.txt        # basic search
grep -i "error" file.txt     # case-insensitive
grep -r "TODO" ./src         # recursive search
grep -c "error" file.txt     # count matches
grep -n "error" file.txt     # show line numbers
grep -v "error" file.txt     # invert match (exclude)
grep -E "error|fail" file.txt # extended regex (multiple patterns)
```

### awk — column/field processing
> 💡 **Real-time use:** Parsing `df -h` or `ps aux` output to pull out just the CPU/memory/disk column for a monitoring script.
```bash
awk '{print $1}' file.txt              # print first column
awk -F: '{print $1}' /etc/passwd       # custom field separator
awk '$3 > 50 {print $1}' data.txt      # pattern-based filter
awk 'BEGIN{print "Start"} {print} END{print "End"}' file.txt
```

### sed — stream editor
> 💡 **Real-time use:** Updating a config file's environment variable (e.g. `APP_ENV=dev` → `APP_ENV=prod`) automatically during deployment.
```bash
sed 's/old/new/' file.txt         # replace first match per line
sed 's/old/new/g' file.txt        # replace all matches
sed '2d' file.txt                 # delete line 2
sed -i 's/foo/bar/g' file.txt     # edit file in-place
```

### cut — extract columns
> 💡 **Real-time use:** Extracting usernames from `/etc/passwd`, or pulling a specific column out of a CSV export for a report.
```bash
cut -d: -f1 /etc/passwd     # extract 1st field, delimiter ":"
cut -d, -f2,3 data.csv       # extract multiple fields
```

### sort
> 💡 **Real-time use:** Sorting access-log IPs numerically to find the most/least active clients, or sorting file sizes to find the biggest disk hogs.
```bash
sort file.txt          # alphabetical
sort -n file.txt        # numerical
sort -r file.txt        # reverse
sort -u file.txt        # unique + sorted
```

### uniq
> 💡 **Real-time use:** Counting how many times each IP hit your server in `access.log` to spot suspicious traffic or a DDoS pattern.
```bash
sort file.txt | uniq          # remove duplicate adjacent lines
sort file.txt | uniq -c       # count occurrences
```

### tr — translate/delete characters
> 💡 **Real-time use:** Cleaning up messy input (removing carriage returns `\r` from Windows-edited files, or normalizing case) before further processing.
```bash
echo "hello" | tr 'a-z' 'A-Z'   # translate to uppercase
echo "hello" | tr -d 'l'         # delete character
```

### wc — word/line/char count
> 💡 **Real-time use:** Quickly checking how many error lines exist in a log file, or validating a file isn't empty/corrupt after a transfer.
```bash
wc -l file.txt   # line count
wc -w file.txt   # word count
wc -c file.txt   # byte/char count
```

### head / tail
> 💡 **Real-time use:** `tail -f` is the #1 command during an incident — live-watching an application log as it grows to catch errors as they happen.
```bash
head -n 10 file.txt    # first 10 lines
tail -n 10 file.txt    # last 10 lines
tail -f app.log        # follow file in real-time (live logs)
```

---

## 6️⃣ Useful Patterns and One-Liners

```bash
# 1. Find and delete files older than 7 days
find /path/to/logs -type f -mtime +7 -exec rm {} \;

# 2. Count total lines across all .log files
wc -l *.log | tail -1

# 3. Replace a string across multiple files
grep -rl "oldstring" ./project | xargs sed -i 's/oldstring/newstring/g'

# 4. Check if a service is running
systemctl is-active --quiet nginx && echo "Running" || echo "Stopped"

# 5. Monitor disk usage and alert if above 80%
df -h / | awk 'NR==2 {gsub("%","",$5); if ($5+0 > 80) print "Alert: Disk usage high!"}'

# 6. Parse a CSV column
awk -F, '{print $2}' data.csv

# 7. Tail a log and filter for errors in real time
tail -f app.log | grep --line-buffered -i "error"
```

---

## 7️⃣ Error Handling and Debugging

### Exit Codes
> 💡 **Real-time use:** A CI/CD pipeline (Jenkins/GitHub Actions) checks the exit code of your script to decide whether the build/deploy step passed or failed.
```bash
echo $?         # exit code of last command (0 = success)
exit 0          # exit script successfully
exit 1          # exit script with error
```

### set -e — Exit on Error
> 💡 **Real-time use:** Deployment scripts use this so that if `npm install` fails, the script doesn't blindly continue to `pm2 restart` on a broken build.
```bash
set -e   # script stops immediately if any command fails
```

### set -u — Error on Unset Variables
> 💡 **Real-time use:** Catches typos like `$DATABSE_URL` instead of `$DATABASE_URL` immediately, instead of silently deploying with a blank config.
```bash
set -u   # using an undefined variable throws an error, not silently blank
```

### set -o pipefail — Catch Errors in Pipes
> 💡 **Real-time use:** Without this, `curl http://badurl | grep "ok"` can report "success" even if `curl` failed — critical for reliable health-check scripts.
```bash
set -o pipefail   # pipeline fails if ANY command in it fails, not just the last
```

### set -x — Debug Mode
> 💡 **Real-time use:** Turned on temporarily when a script fails in staging and you need to see exactly which command and variable value caused the failure.
```bash
set -x   # prints each command before executing it (great for debugging)
set +x   # turn off debug mode
```

### trap — Cleanup on Exit
> 💡 **Real-time use:** Ensuring temporary files, lock files, or SSH tunnels are always removed/closed — even if the script crashes midway.
```bash
cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT   # runs 'cleanup' automatically when script exits
```

### Recommended Safety Header
```bash
#!/bin/bash
set -euo pipefail
```

---

## 8️⃣ Real-Time DevOps Scripts (Practical Examples)

These are complete, ready-to-use scripts you'd actually write on the job — not just one-liners.

### 🔹 Server Health Check Script
Checks CPU, memory, and disk usage, and alerts if any goes above a threshold.
```bash
#!/bin/bash
set -euo pipefail

THRESHOLD=80

CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM=$(free | awk '/Mem/{printf("%.0f"), $3/$2*100}')
DISK=$(df -h / | awk 'NR==2{gsub("%","",$5); print $5}')

echo "CPU: $CPU% | MEM: $MEM% | DISK: $DISK%"

if (( $(echo "$CPU > $THRESHOLD" | bc -l) )); then
    echo "⚠️  ALERT: High CPU usage!"
fi
if [ "$DISK" -gt "$THRESHOLD" ]; then
    echo "⚠️  ALERT: Disk usage above ${THRESHOLD}%!"
fi
```
**Why:** Same logic (awk parsing, if-conditions, thresholds) used in real monitoring/alerting cron jobs.

---

### 🔹 Automated Backup Script
Backs up a directory with a timestamp, keeps only the last 5 backups.
```bash
#!/bin/bash
set -euo pipefail

SRC_DIR="/var/www/myapp"
BACKUP_DIR="/backups"
DATE=$(date +%F_%H-%M-%S)
BACKUP_FILE="$BACKUP_DIR/myapp_backup_$DATE.tar.gz"

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_FILE" "$SRC_DIR"
echo "Backup created: $BACKUP_FILE"

# keep only last 5 backups, delete older ones
cd "$BACKUP_DIR"
ls -tp myapp_backup_*.tar.gz | grep -v '/$' | tail -n +6 | xargs -I {} rm -- {}
```
**Why:** Combines variables, `date`, `tar`, and `ls`/`xargs` — a real backup rotation strategy used in production servers.

---

### 🔹 Service Auto-Restart Watchdog
Checks if a service (e.g. nginx) is down, and restarts it automatically.
```bash
#!/bin/bash
SERVICE="nginx"

if ! systemctl is-active --quiet "$SERVICE"; then
    echo "$(date): $SERVICE is down. Restarting..." >> /var/log/watchdog.log
    systemctl restart "$SERVICE"
else
    echo "$(date): $SERVICE is running fine." >> /var/log/watchdog.log
fi
```
**Why:** Run this via a cron job every 5 minutes — a simple but real self-healing setup for critical services.

---

### 🔹 Log Cleanup + Error Report Script
Deletes logs older than 30 days and emails/prints a summary of errors found today.
```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
TODAY=$(date +%F)

# delete logs older than 30 days
find "$LOG_DIR" -name "*.log" -type f -mtime +30 -delete

# count today's errors across all logs
ERROR_COUNT=$(grep -ri "error" "$LOG_DIR"/*.log 2>/dev/null | grep "$TODAY" | wc -l)

echo "Error report for $TODAY: $ERROR_COUNT errors found"
```
**Why:** Real log-hygiene + reporting task — common in scheduled maintenance scripts.

---

### 🔹 Deployment Script (Simple CI/CD Style)
Pulls latest code, installs dependencies, restarts the app — with rollback on failure.
```bash
#!/bin/bash
set -euo pipefail

APP_DIR="/var/www/myapp"
cd "$APP_DIR"

echo "Pulling latest code..."
git pull origin main

echo "Installing dependencies..."
npm install

echo "Restarting application..."
if ! pm2 restart myapp; then
    echo "Deployment failed! Rolling back..."
    git reset --hard HEAD~1
    pm2 restart myapp
    exit 1
fi

echo "✅ Deployment successful"
```
**Why:** Shows `set -e`, `git`, conditionals, and rollback logic together — the core idea behind real deployment scripts before you move to Jenkins/GitHub Actions.

---

## 📚 Summary

This cheat sheet covers Bash basics, conditionals, loops, functions, text processing tools, real-world one-liners, error handling/debugging, and complete real-time scripts (health check, backup, watchdog, log cleanup, deployment) — everything needed as a quick reference for daily DevOps scripting work.


