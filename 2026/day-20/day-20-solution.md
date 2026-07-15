# Day 20 – Bash Scripting Challenge: Log Analyzer and Report Generator

## 📌 Project Overview

### Objective

The objective of this project is to build a Bash script that automates log file analysis. The script validates user input, analyzes server logs, identifies errors and critical events, generates a summary report, and archives processed log files. This project demonstrates how Bash scripting can automate repetitive system administration tasks.

### Real-World Scenario

In a production environment, servers generate thousands of log entries every day. It is not practical for a DevOps Engineer or System Administrator to manually inspect every log file. An automated log analyzer helps quickly identify errors, detect critical issues, generate reports for troubleshooting, and archive processed logs to keep the working directory organized.

---

# Task 1 – Input Validation

## 🎯 Objective

Validate the user input before starting the log analysis.

## ❓What is this Task?

Before processing the log file, the script checks:

* Whether the user has provided a log file as an argument.
* Whether the specified log file exists.

If any validation fails, the script displays an appropriate error message and exits safely.

---

## ❓Why is it Important?

Input validation prevents the script from processing invalid input.

Benefits:

* Prevents unexpected runtime errors.
* Improves script reliability.
* Makes troubleshooting easier.
* Follows production scripting best practices.

---

## 🌍 Real-World Use Case

Before executing backup, deployment, or monitoring scripts, DevOps Engineers always validate user input. Running a script with an incorrect file path could lead to failures or incorrect results.

---

## 💻 Commands Used

| Command  | Meaning                                                     |
| -------- | ----------------------------------------------------------- |
| `$#`     | Returns the total number of arguments passed to the script. |
| `$1`     | Represents the first command-line argument.                 |
| `$0`     | Represents the script name.                                 |
| `-f`     | Checks whether the file exists and is a regular file.       |
| `!`      | Negates the condition (NOT operator).                       |
| `exit 1` | Stops the script with a failure status code.                |

---

## 📝 Script

```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Error: No log file provided."
    echo "Usage: $0 <log-file>"
    exit 1
fi

LOG_FILE="$1"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: File '$LOG_FILE' does not exist."
    exit 1
fi

echo "Log file '$LOG_FILE' found. Proceeding with analysis..."
```

---

## 💻 Output

```text
$ ./input_validation.sh
Error: No log file provided.

$ ./input_validation.sh wrong.log
Error: File 'wrong.log' does not exist.

$ ./input_validation.sh sample_log.log
Log file 'sample_log.log' found. Proceeding with analysis...
```

---

## 📷 Screenshot

![Task 1](1.Input%20and%20Validation.png)

---

## 📖 Explanation

This task ensures that only valid input is processed. By validating the file before analysis, the script avoids unnecessary failures and behaves like a production-ready automation script.

---

# Task 2 – Error Count

## 🎯 Objective

Count the total number of ERROR and Failed entries in the log file.

---

## ❓What is this Task?

This task scans the entire log file and counts all lines containing either **ERROR** or **Failed**.

---

## ❓Why is it Important?

Instead of manually searching the log file, administrators can immediately understand how many errors occurred.

---

## 🌍 Real-World Use Case

Monitoring tools frequently count errors to determine the health of an application or server.

---

## 💻 Commands Used

| Command         | Meaning                                  |
| --------------- | ---------------------------------------- |
| `grep`          | Searches for matching text inside files. |
| `grep -c`       | Counts matching lines.                   |
| `grep -E`       | Enables extended regular expressions.    |
| `ERROR\|Failed` | Matches either ERROR or Failed.          |
| `$(...)`        | Stores command output inside a variable. |

---

## 📝 Script

```bash
ERROR_COUNT=$(grep -cE "ERROR|Failed" "$LOG_FILE")

echo "Total Error Count: $ERROR_COUNT"
```

---

## 💻 Output

```text
Total Error Count: 10
```

---

## 📷 Screenshot

![Task 2](2.Error%20Count.png)

---

## 📖 Explanation

The total error count provides a quick overview of the log's health. Instead of reading every log entry, administrators can immediately determine whether further investigation is needed.

---

# Task 3 – Critical Events

## 🎯 Objective

Identify all CRITICAL events along with their line numbers.

---

## ❓What is this Task?

The script searches for every CRITICAL log entry and displays the exact line number.

---

## ❓Why is it Important?

Critical issues require immediate attention. Displaying the line number helps engineers quickly locate the issue in large log files.

---

## 🌍 Real-World Use Case

Critical events such as database failures or disk space issues are usually escalated immediately in production environments.

---

## 💻 Commands Used

| Command   | Meaning                                      |
| --------- | -------------------------------------------- |
| `grep -n` | Displays matching lines with line numbers.   |
| `sed -E`  | Modifies text using regular expressions.     |
| `\1`      | Refers to the captured group in the pattern. |

---

## 📝 Script

```bash
echo "--- Critical Events ---"

grep -n "CRITICAL" "$LOG_FILE" | sed -E 's/^([0-9]+):/Line \1: /'
```

---

## 💻 Output

```text
Line 5: CRITICAL Disk space below threshold

Line 9: CRITICAL Database connection lost
```

---

## 📷 Screenshot

![Task 3](3.Critical%20Events.png)

---

## 📖 Explanation

Adding line numbers allows engineers to directly locate critical events without manually searching through the log file.

---

# Task 4 – Top 5 Error Messages

## 🎯 Objective

Find the most frequently occurring error messages.

---

## ❓What is this Task?

The script extracts all ERROR entries, removes unnecessary fields, groups identical messages, counts their occurrences, and displays the Top 5.

---

## ❓Why is it Important?

Knowing which error occurs most frequently helps prioritize troubleshooting.

---

## 🌍 Real-World Use Case

Monitoring dashboards often display the most common errors to help engineers focus on the highest-impact problems.

---

## 💻 Commands Used

| Command    | Meaning                             |
| ---------- | ----------------------------------- |
| `awk`      | Processes text column by column.    |
| `sort`     | Sorts data alphabetically.          |
| `uniq -c`  | Counts duplicate entries.           |
| `sort -rn` | Sorts counts in descending order.   |
| `head -5`  | Displays only the top five results. |

---

## 📝 Script

```bash
TOP_ERRORS=$(grep "ERROR" "$LOG_FILE" \
| awk '{$1=$2=$3=""; print}' \
| sort \
| uniq -c \
| sort -rn \
| head -5)

echo "$TOP_ERRORS"
```

---

## 💻 Output

```text
3 Connection timed out

2 Permission denied

1 Out of memory

1 File not found

1 Disk I/O error
```

---

## 📷 Screenshot

![Task 4](4.Top%20Error%20Messages.png)

---

## 📖 Explanation

This task highlights recurring issues, making it easier to identify the primary cause of system failures.

---

# Task 5 – Generate Summary Report

## 🎯 Objective

Generate a daily log analysis report.

---

## ❓What is this Task?

The script creates a report containing:

* Date
* Log file name
* Total lines
* Error count
* Top errors
* Critical events

---

## ❓Why is it Important?

Reports can be stored, shared, and compared over time for troubleshooting and auditing.

---

## 🌍 Real-World Use Case

Most enterprise monitoring systems automatically generate daily reports for operations teams.

---

## 💻 Commands Used

| Command | Meaning                       |
| ------- | ----------------------------- |
| `date`  | Gets the current date.        |
| `wc -l` | Counts total lines.           |
| `>`     | Creates or overwrites a file. |
| `>>`    | Appends content to a file.    |

---

## 📝 Script

```bash
TODAY=$(date +%Y-%m-%d)

REPORT_FILE="log_report_${TODAY}.txt"

TOTAL_LINES=$(wc -l < "$LOG_FILE")
```

(Continue writing report contents using `echo >> "$REPORT_FILE"`.)

---

## 💻 Output

```text
Report generated: log_report_2026-07-15.txt
```

---

## 📷 Screenshot

![Task 5](5.Summary%20Report.png)

---

## 📖 Explanation

The report provides a permanent record of log analysis, making future review and troubleshooting much easier.

---

# Task 6 – Archive Processed Logs

## 🎯 Objective

Move processed log files into an archive directory.

---

## ❓What is this Task?

After successful analysis, the processed log file is moved into an archive folder.

---

## ❓Why is it Important?

Archiving keeps the working directory clean and prevents the same log file from being processed multiple times.

---

## 🌍 Real-World Use Case

Production systems archive processed logs daily for better organization and storage management.

---

## 💻 Commands Used

| Command    | Meaning                                              |
| ---------- | ---------------------------------------------------- |
| `mkdir -p` | Creates the archive directory if it does not exist.  |
| `mv`       | Moves the processed file into the archive directory. |

---

## 📝 Script

```bash
mkdir -p archive

mv "$LOG_FILE" archive/

echo "Log file archived to archive/$LOG_FILE"
```

---

## 💻 Output

```text
Log file archived to archive/sample_log.log
```

---

## 📷 Screenshot

![Task 6](6.Archive%20Processed%20Logs.png)

---

## 📖 Explanation

Archiving processed logs helps maintain a clean workspace and prevents duplicate analysis of the same log file.

---

# 🐞 Mistakes I Faced During This Project

## Issue 1 – Duplicate "Top 5 Error Messages"

### Problem

The Top 5 Error Messages section was printed twice.

### Root Cause

I accidentally kept both the original pipeline and the new variable-based implementation.

### Fix

I removed the duplicate pipeline and stored the result only in the `TOP_ERRORS` variable.

---

## Issue 2 – File Does Not Exist

### Problem

```text
Error: File 'sample_log.log' does not exist.
```

### Root Cause

The `mv` command moved the file into the `archive/` directory after processing.

### Fix

```bash
cp archive/sample_log.log .

./input_validation.sh sample_log.log
```

or recreate the sample log file before testing again.

---

# 📚 Key Learnings

* Learned how to validate user input in Bash.
* Used grep, awk, sed, sort, uniq, and head for log analysis.
* Generated automated reports using Bash.
* Learned command substitution using `$(...)`.
* Understood file redirection using `>` and `>>`.
* Learned how to archive processed log files.
* Improved debugging and troubleshooting skills.

---

# 🛠 Commands Used Throughout the Project

* grep
* awk
* sed
* sort
* uniq
* head
* wc
* date
* mkdir
* mv
* cp
* echo
* if
* exit
* command substitution `$(...)`
* output redirection (`>`, `>>`)

---

# ✅ Conclusion

This project helped me understand how Bash scripting can automate log analysis tasks commonly performed by DevOps Engineers and System Administrators. By combining input validation, text processing, report generation, and log archiving, I built a practical automation script that reflects real-world production workflows and strengthened my Linux command-line and debugging skills.
