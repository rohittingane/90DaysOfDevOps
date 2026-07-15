# Day 19 – Mini Project: Server Maintenance Automation

# Objective

Today I built a mini DevOps automation project using Bash scripting. I combined multiple scripts into a complete server maintenance workflow that performs log rotation, server backups, maintenance logging, and task scheduling using Cron. This project demonstrates how repetitive server administration tasks can be automated in a real-world Linux environment.

---

# Task 1 – Log Rotation Script

## Goal

Create a script that automatically manages old log files by compressing and deleting them based on their age.

## Script

**log_rotate.sh**

## Features

* Compresses `.log` files older than 7 days.
* Deletes compressed `.log.gz` files older than 30 days.
* Displays the number of compressed and deleted files.
* Accepts the log directory as a command-line argument.
* Provides success messages after completion.

## Commands Used

```bash
chmod +x log_rotate.sh
./log_rotate.sh mylogs/
find
gzip
```

## Observation

* Successfully compressed old log files.
* Deleted compressed log files older than 30 days.
* Displayed the total number of compressed and deleted files.
* Automated log cleanup without manual intervention.

## Output

![Log Rotation Script Execution](./1.log-rotation-success.png)

---

# Task 2 – Server Backup Script

## Goal

Create a reusable backup script that archives important directories and automatically removes old backups.

## Script

**backup.sh**

## Features

* Accepts source and backup destination directories as arguments.
* Creates a timestamped `.tar.gz` archive.
* Verifies successful archive creation.
* Displays archive name and archive size.
* Deletes backup files older than 14 days.
* Exits with an error if the source directory does not exist.

## Commands Used

```bash
chmod +x backup.sh
./backup.sh test_data backups
tar -czf
find
date
```

## Observation

* Successfully created a compressed backup archive.
* Verified backup file creation.
* Displayed archive name and size.
* Automatically removed expired backup files.

## Output

![Server Backup Script Execution](./2-server-backup-script.png)

---

# Task 3 – Cron Scheduling

## Goal

Understand how Cron automates recurring tasks in Linux.

## Current Cron Jobs

```bash
crontab -l
```

## Existing Cron Job

```cron
* * * * * cd /home/ubuntu/my-app && git pull origin main && pm2 restart app
```

## Sample Cron Entries

### Run Log Rotation every day at 2 AM

```cron
0 2 * * * /home/ubuntu/log_rotate.sh /home/ubuntu/mylogs
```

### Run Backup every Sunday at 3 AM

```cron
0 3 * * 0 /home/ubuntu/backup.sh /home/ubuntu/test_data /home/ubuntu/backups
```

### Run Health Check every 5 minutes

```cron
*/5 * * * * /home/ubuntu/health_check.sh
```

## Observation

* Learned Cron syntax.
* Understood minute, hour, day, month, and weekday fields.
* Verified the existing cron job using `crontab -l`.
* Prepared sample cron schedules for server automation.

## Output

![Crontab Configuration](./3-crontab-output.png)

---

# Task 4 – Scheduled Maintenance Script

## Goal

Create a master maintenance script that combines log rotation and backup into a single automated workflow.

## Script

**maintenance.sh**

## Features

* Calls `log_rotate.sh`
* Calls `backup.sh`
* Logs all output to `/var/log/maintenance.log`
* Adds timestamps for every operation
* Displays maintenance execution details

## Cron Entry

Run every day at **1 AM**

```cron
0 1 * * * /home/ubuntu/maintenance.sh
```

## Commands Used

```bash
chmod +x maintenance.sh
sudo ./maintenance.sh
cat /var/log/maintenance.log
```

## Observation

* Successfully executed log rotation.
* Successfully executed backup.
* Logged maintenance activity with timestamps.
* Verified maintenance log contents.

## Output

![Scheduled Maintenance Script Execution](./4-maintenance-script-output.png)

---

# Commands Used

```bash
crontab -l
crontab -e
tar -czf
find
gzip
date
cat /var/log/maintenance.log
chmod +x
```

---

# Screenshots

| Screenshot | Description |
| -------------------------------- | ------------------------------------------------------------------- |
| 1.log-rotation-success.png | Successfully compressed old log files and deleted expired archives. |
| 2-server-backup-script.png | Created timestamped backup archive and displayed archive size. |
| 3-crontab-output.png | Displayed current cron jobs and prepared automation schedules. |
| 4-maintenance-script-output.png | Executed maintenance script and logged output with timestamps. |

---

# Skills Practiced

* Bash Scripting
* Linux File Management
* TAR Archive Creation
* Log Rotation
* Backup Automation
* Error Handling
* Command-Line Arguments
* Cron Scheduling
* Linux Automation
* Server Maintenance

---

# What I Learned

* Automated server maintenance tasks using Bash scripts.
* Created reusable scripts using command-line arguments.
* Compressed and managed log files automatically.
* Created timestamped backup archives.
* Scheduled recurring tasks using Cron.
* Combined multiple scripts into a single maintenance workflow.
* Logged maintenance activities for monitoring and troubleshooting.

---

# Conclusion

Today's mini project helped me understand how DevOps engineers automate routine server maintenance tasks. Instead of manually rotating logs, creating backups, and monitoring maintenance activities, I combined everything into a single automated workflow using Bash scripting and Cron. This project strengthened my understanding of Linux automation, scripting, backup management, and scheduled task execution, making it closer to real-world DevOps practices.
