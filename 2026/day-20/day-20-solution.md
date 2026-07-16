# Day 20 – Bash Scripting Challenge: Log Analyzer and Report Generator

## 📌 Project Overview

### Objective

The objective of this project is to build a Bash script that automates log file analysis. The script validates user input, analyzes server logs, identifies errors and critical events, generates a summary report, and archives processed log files.

This project demonstrates how Bash scripting can automate repetitive system administration tasks and reflects a real-world DevOps automation workflow.

---

### 🌍 Real-World Scenario

In production environments, servers generate thousands of log entries every day. Manually reviewing these logs is time-consuming and inefficient.

A Log Analyzer helps DevOps Engineers and System Administrators by:

- Validating user input before execution
- Detecting errors and critical events
- Identifying the most frequent issues
- Generating a summary report
- Archiving processed logs for better organization

This automation reduces manual effort and speeds up troubleshooting.

---

# Task 1 – Input Validation

## 🎯 Objective

Validate the user input before starting the log analysis.

---

## ❓What is this Task?

Before processing the log file, the script checks:

- Whether the user has provided a log file as an argument.
- Whether the specified log file exists.

If either validation fails, the script displays an appropriate error message and exits safely.

---

## ❓Why is it Important?

Input validation prevents the script from processing invalid input.

Benefits:

- Prevents unexpected runtime errors
- Improves script reliability
- Makes troubleshooting easier
- Follows production scripting best practices

---

## 🌍 Real-World Use Case

Before running backup, deployment, or monitoring scripts, DevOps Engineers always validate user input. Executing a script with an incorrect file path can lead to failures or inaccurate results.

---

## 💻 Commands Used

| Command | Purpose |
|----------|---------|
| `$#` | Returns the total number of arguments |
| `$1` | Represents the first command-line argument |
| `$0` | Represents the script name |
| `-f` | Checks whether the file exists |
| `!` | Negates the condition |
| `exit 1` | Exits the script with an error status |

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

The screenshot below demonstrates the input validation process. It shows how the script handles missing arguments, invalid file names, and successfully validates an existing log file before starting the analysis.

![Task 1 - Input Validation](1.Input%20and%20Validation.png)

---

## 📖 Explanation

This task ensures that only valid input is processed. By validating the file before analysis, the script avoids unnecessary failures and behaves like a production-ready automation script.

---

# Task 2 – Error Count

## 🎯 Objective

Count the total number of **ERROR** and **Failed** entries in the log file.

---

## ❓What is this Task?

The script scans the entire log file and counts every line containing either **ERROR** or **Failed**.

---

## ❓Why is it Important?

Instead of manually searching through a large log file, administrators can quickly determine how many errors occurred.

---

## 🌍 Real-World Use Case

Monitoring tools continuously count application and server errors to measure overall system health and trigger alerts when necessary.

---

## 💻 Commands Used

| Command | Purpose |
|----------|---------|
| `grep` | Searches text inside files |
| `grep -c` | Counts matching lines |
| `grep -E` | Enables extended regular expressions |
| `ERROR\|Failed` | Matches ERROR or Failed |
| `$(...)` | Stores command output inside a variable |

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

The screenshot below displays the total number of **ERROR** and **Failed** entries identified during log analysis.

![Task 2 - Error Count](2.Error%20Count.png)

---

## 📖 Explanation

The total error count provides a quick overview of the health of the log file. This allows administrators to decide whether further investigation is required.


# Task 3 – Critical Events

## 🎯 Objective

Identify all **CRITICAL** events along with their line numbers.

---

## ❓What is this Task?

This task scans the log file and searches for every **CRITICAL** event. Along with the log message, it also displays the corresponding line number so that the issue can be located quickly.

---

## ❓Why is it Important?

Critical events usually indicate serious problems that require immediate attention. Displaying line numbers saves time by allowing engineers to jump directly to the affected log entry instead of searching manually.

Benefits:

- Quickly locate critical issues
- Reduce troubleshooting time
- Improve incident response
- Simplify log analysis

---

## 🌍 Real-World Use Case

In production environments, critical events such as application crashes, database failures, or low disk space are immediately investigated by DevOps Engineers. Displaying the line number helps teams locate and resolve the issue faster.

---

## 💻 Commands Used

| Command | Purpose |
|----------|---------|
| `grep -n` | Searches for matching text and displays line numbers |
| `sed -E` | Formats the output using regular expressions |
| `\1` | References the captured line number |

---

## 📝 Script

```bash
echo "--- Critical Events ---"

grep -n "CRITICAL" "$LOG_FILE" | sed -E 's/^([0-9]+):/Line \1: /'
```

---

## 💻 Output

```text
--- Critical Events ---

Line 5: CRITICAL Disk space below threshold

Line 9: CRITICAL Database connection lost
```

---

## 📷 Screenshot

The screenshot below shows all **CRITICAL** events detected in the log file along with their corresponding line numbers.

![Task 3 - Critical Events](3.Critical%20Events.png)

---

## 📖 Explanation

By displaying line numbers with each critical event, the script makes troubleshooting much faster. Engineers can directly navigate to the affected log entry without manually searching through the entire file.

---

# Task 4 – Top 5 Error Messages

## 🎯 Objective

Find the **Top 5** most frequently occurring error messages.

---

## ❓What is this Task?

The script extracts all **ERROR** log entries, removes unnecessary fields, groups identical messages together, counts their occurrences, and displays the five most common errors.

---

## ❓Why is it Important?

Knowing which error occurs most frequently helps administrators prioritize troubleshooting efforts and identify recurring issues affecting the system.

Benefits:

- Identifies recurring problems
- Helps prioritize fixes
- Reduces troubleshooting effort
- Improves system reliability

---

## 🌍 Real-World Use Case

Monitoring platforms often display the most common application errors on dashboards. This allows DevOps teams to focus first on the issues affecting the largest number of users or services.

---

## 💻 Commands Used

| Command | Purpose |
|----------|---------|
| `grep` | Filters ERROR log entries |
| `awk` | Removes unwanted columns |
| `sort` | Sorts the extracted messages |
| `uniq -c` | Counts duplicate messages |
| `sort -rn` | Sorts counts in descending order |
| `head -5` | Displays only the top five results |

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

The screenshot below displays the **Top 5** most frequently occurring error messages identified from the log file.

![Task 4 - Top Error Messages](4.Top%20Error%20Messages.png)

---

## 📖 Explanation

This task helps identify recurring issues by highlighting the most common error messages. Instead of checking every log entry manually, administrators can immediately focus on the errors occurring most frequently.


# Task 5 – Generate Summary Report

## 🎯 Objective

Generate a daily log analysis report containing key information from the processed log file.

---

## ❓What is this Task?

After completing the log analysis, the script automatically creates a summary report that includes:

- Date of analysis
- Log file name
- Total number of log entries
- Total error count
- Top 5 error messages
- Critical events

The report is saved as a text file for future reference.

---

## ❓Why is it Important?

Instead of checking the log file repeatedly, administrators can review a summarized report containing all important information.

Benefits:

- Saves analysis results
- Simplifies troubleshooting
- Makes reporting easier
- Helps maintain historical records
- Supports auditing and monitoring

---

## 🌍 Real-World Use Case

Most enterprise monitoring tools automatically generate daily reports for operations teams. These reports help engineers review system health, investigate incidents, and maintain audit records.

---

## 💻 Commands Used

| Command | Purpose |
|----------|---------|
| `date` | Gets the current date |
| `wc -l` | Counts total lines in the log file |
| `>` | Creates or overwrites a file |
| `>>` | Appends data to a file |

---

## 📝 Script

```bash
TODAY=$(date +%Y-%m-%d)

REPORT_FILE="log_report_${TODAY}.txt"

TOTAL_LINES=$(wc -l < "$LOG_FILE")

# Report content is written using echo >> "$REPORT_FILE"
```

---

## 💻 Output

```text
Report generated: log_report_2026-07-15.txt
```

---

## 📷 Screenshot

The screenshot below shows the generated summary report containing the log analysis details, including total log entries, error count, top error messages, and critical events.

![Task 5 - Summary Report](5.Summary%20Report.png)

---

## 📖 Explanation

Generating a summary report provides a permanent record of log analysis. It helps administrators quickly review important information without reopening and analyzing the original log file.

---

# Task 6 – Archive Processed Logs

## 🎯 Objective

Move the processed log file into an archive directory after successful analysis.

---

## ❓What is this Task?

Once the analysis is complete, the script automatically creates an **archive** directory (if it doesn't already exist) and moves the processed log file into it.

---

## ❓Why is it Important?

Archiving processed logs keeps the working directory clean and prevents the same log file from being analyzed multiple times.

Benefits:

- Keeps the workspace organized
- Prevents duplicate processing
- Simplifies log management
- Improves storage organization

---

## 🌍 Real-World Use Case

Production servers often archive processed logs daily or weekly to improve storage management and simplify future troubleshooting.

---

## 💻 Commands Used

| Command | Purpose |
|----------|---------|
| `mkdir -p` | Creates the archive directory if it does not exist |
| `mv` | Moves the processed log file into the archive directory |

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

The screenshot below shows the successful archiving of the processed log file into the **archive** directory.

![Task 6 - Archive Processed Logs](6.Archive%20Processed%20Logs.png)

---

## 📖 Explanation

Archiving processed logs helps maintain a clean working directory and ensures that the same log file is not processed repeatedly.

---

# 🐞 Mistakes I Faced During This Project

## Issue 1 – Duplicate "Top 5 Error Messages"

### ❌ Problem

The **Top 5 Error Messages** section was displayed twice in the output.

### 🔍 Root Cause

I accidentally kept both the original command pipeline and the variable-based implementation, causing duplicate output.

### ✅ Solution

I removed the duplicate command pipeline and stored the output only in the `TOP_ERRORS` variable before printing it.

---

## Issue 2 – File Does Not Exist

### ❌ Problem

```text
Error: File 'sample_log.log' does not exist.
```

### 🔍 Root Cause

The script successfully moved the processed log file into the **archive** directory, so it was no longer available in the current working directory during the next execution.

### ✅ Solution

I copied the file back from the archive directory before testing again.

```bash
cp archive/sample_log.log .

./log_analyzer.sh sample_log.log
```

Alternatively, the sample log file can be recreated before rerunning the script.

---

# 📚 Key Learnings

During this project, I learned how Bash scripting can automate real-world log analysis tasks commonly performed by DevOps Engineers and System Administrators.

### Skills Gained

- Validated user input before processing
- Used `grep` to search and count log entries
- Extracted and formatted data using `awk` and `sed`
- Counted duplicate entries using `sort` and `uniq`
- Generated automated reports
- Archived processed log files
- Improved debugging and troubleshooting skills
- Built a practical Bash automation project

---

# 🛠 Commands Used Throughout the Project

- grep
- awk
- sed
- sort
- uniq
- head
- wc
- date
- mkdir
- mv
- cp
- echo
- if
- exit
- command substitution `$(...)`
- output redirection (`>` and `>>`)

---

# ✅ Conclusion

This project helped me understand how Bash scripting can automate real-world log analysis tasks. By combining input validation, log parsing, text processing, report generation, and log archiving, I built a practical automation script that reflects common DevOps workflows.

Working on this project also strengthened my Linux command-line skills, improved my understanding of Bash scripting, and gave me hands-on experience with troubleshooting production-style log files.
