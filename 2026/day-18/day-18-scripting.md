# Day 18 – Shell Scripting: Functions & Intermediate Concepts

## 📖 Overview

Today I continued learning Shell Scripting and explored intermediate concepts used in real-world DevOps automation.

I learned how to write reusable scripts using functions, improve script safety using strict mode (`set -euo pipefail`), work with local variables, and build a system information reporting script.

---

# Task 1: Basic Functions

## Script: functions.sh

```bash
#!/bin/bash

greet() {
    echo "Hello Rohit!"
}

greet


add() {
    echo $(($1+$2))
}


add 10 20
```

## Output

```text
Hello Rohit!
30
```

## Explanation

Functions allow us to create reusable blocks of code.

Syntax:

```bash
function_name() {
    commands
}
```

### greet()

The `greet` function prints a greeting message.

### add()

The `add` function takes two arguments:

- `$1` → First argument
- `$2` → Second argument

Example:

```bash
add 10 20
```

Output:

```text
30
```

## Screenshot

![Task 1 - Functions](./1-functions.png)

---

# Task 2: Functions with Disk and Memory Check

## Script: disk_check.sh

```bash
#!/bin/bash

check_disk() {
    echo "Checking Disk Usage..."
    df -h /
}


check_memory() {
    echo "Checking Memory Usage..."
    free -h
}


check_disk

check_memory
```

## Output

```text
Checking Disk Usage...

Filesystem      Size  Used Avail Use% Mounted on
/dev/root       6.8G  5.1G  1.7G  76% /

Checking Memory Usage...

               total        used        free
Mem:           911Mi       430Mi       118Mi
Swap:             0B          0B          0B
```

## Explanation

Created separate functions for:

- Checking disk usage
- Checking memory usage

Functions make scripts cleaner and reusable.

## Screenshot

![Task 2 - Disk Memory Check](./2-disk-memory-check.png)

---

# Task 3: Strict Mode – set -euo pipefail

## Script: strict_demo.sh

```bash
#!/bin/bash

set -euo pipefail

echo "Starting Test"

false | true

echo "Script End"
```

## Output

```text
Starting Test
```

## Explanation

Strict mode helps to create safer Bash scripts.

---

## set -e

Stops the script immediately when any command fails.

Example:

```bash
cd abc
```

If the directory does not exist, script stops.

---

## set -u

Stops the script when using undefined variables.

Example:

```bash
echo $NAME
```

If variable is not defined:

```text
NAME: unbound variable
```

---

## set -o pipefail

Detects failures inside pipelines.

Example:

```bash
false | true
```

With pipefail, pipeline failure is detected.

## Screenshot

![Task 3 - Strict Mode Output](./3-strict-mode-output.png)

---

# Task 4: Local Variables

## Script: local_demo.sh

### Using local keyword

```bash
#!/bin/bash

demo() {

    local NAME="Rohit"

    echo "Inside Function: $NAME"
}

demo

echo "Outside Function: $NAME"
```

## Output

```text
Inside Function: Rohit
Outside Function:
```

---

## Without local keyword

```bash
#!/bin/bash

demo() {

    NAME="Rohit"

    echo "Inside Function: $NAME"
}

demo

echo "Outside Function: $NAME"
```

## Output

```text
Inside Function: Rohit
Outside Function: Rohit
```

## Explanation

### Local Variable

- Available only inside the function
- Prevents accidental changes in other parts of the script

### Global Variable

- Available throughout the script
- Can affect other functions

## Screenshot

![Task 4 - Local Variable Demo](./4-local-variable-demo.png)

---

# Task 5: System Info Reporter

## Script: system_info.sh

```bash
#!/bin/bash

set -euo pipefail

system_info() {
    echo "========================================"
    echo "       SYSTEM INFORMATION REPORT"
    echo "========================================"
    echo "Hostname: $(hostname)"
    echo "OS: $(grep PRETTY_NAME /etc/os-release | cut -d= -f2 | tr -d '"')"
    echo
}

show_uptime() {
    echo "========== UPTIME =========="
    uptime
    echo
}

disk_usage() {
    echo "========== DISK USAGE (Top 5) =========="
    df -h | head -n 6
    echo
}

memory_usage() {
    echo "========== MEMORY USAGE =========="
    free -h
    echo
}

top_processes() {
    echo "========== TOP 5 CPU PROCESSES =========="
    ps -eo pid,comm,%cpu --sort=-%cpu | head -n 6
    echo
}


main() {

    system_info
    show_uptime
    disk_usage
    memory_usage
    top_processes

}


main
```

## Output

```text
========================================
       SYSTEM INFORMATION REPORT
========================================
Hostname: ip-172-31-36-59
OS: Ubuntu 24.04.4 LTS

========== UPTIME ==========
up 7 days

========== DISK USAGE ==========
Filesystem Size Used Avail Use%

========== MEMORY USAGE ==========
Mem:

========== TOP 5 CPU PROCESSES ==========
PID COMMAND %CPU
```

## Explanation

This script collects basic system information:

- Hostname and OS details
- Server uptime
- Disk usage
- Memory usage
- Top CPU consuming processes

It uses functions to keep the script clean and reusable.

## Screenshot

![Task 5 - System Info Report](./5-system-info-report.png)

---

# What I Learned Today

✅ How to create and call Bash functions  
✅ How to use arguments inside functions  
✅ How to use `set -euo pipefail` for safer scripts  
✅ Difference between local and global variables  
✅ How to build a real-world system information reporting script  

---

# Key Takeaways

Shell scripting is an important skill for DevOps engineers.

Using functions, strict mode, and reusable patterns helps us write cleaner, safer, and production-ready automation scripts.

