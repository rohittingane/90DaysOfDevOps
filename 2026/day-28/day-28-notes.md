# Day 28 - DevOps Revision Day 🚀

## Overview

Day 28 was a revision day to review my complete DevOps learning journey from Day 1 to Day 27.

The main focus was to revise important concepts, identify weak areas, and improve confidence through hands-on practice.

During this revision, I covered:

- Linux Administration
- Shell Scripting
- Networking Fundamentals
- Git & GitHub Workflows
- GitHub CLI

---

# 1. Linux Administration Revision 🐧

Linux is one of the most important skills for a DevOps engineer.

During my Linux revision, I practiced file management, process management, services, permissions, ownership, and system troubleshooting.

---

## File System Navigation

Linux follows a hierarchical file system structure.

Important directories:

| Directory | Purpose |
|---|---|
| / | Root directory |
| /etc | Configuration files |
| /var | Logs and variable data |
| /home | User home directories |
| /tmp | Temporary files |
| /bin | Essential commands |

---

## Basic File Commands

### Check current location

```bash
pwd
```

### List files

```bash
ls
```

### Create files

```bash
touch file.txt
```

### Create directory

```bash
mkdir folder-name
```

### Copy files

```bash
cp source destination
```

### Move files

```bash
mv source destination
```

### Remove files

```bash
rm file.txt
```

---

# Process Management

A process is a running instance of a program.

Monitoring processes helps identify performance issues.

## View running processes

```bash
ps aux
```

## Real-time process monitoring

```bash
top
```

## Stop a process

```bash
kill <PID>
```

---

# Systemd Service Management

Systemd manages services running in Linux.

Examples:

- Start service
- Stop service
- Restart service
- Check service status


## Check service status

```bash
systemctl status nginx
```

## Start service

```bash
sudo systemctl start nginx
```

## Stop service

```bash
sudo systemctl stop nginx
```

## Enable service at boot

```bash
sudo systemctl enable nginx
```

---

# File Permissions

Linux permissions control access to files and directories.

There are three types of users:

- Owner
- Group
- Others

Permission types:

| Permission | Meaning |
|---|---|
| r | Read |
| w | Write |
| x | Execute |


Example:

```bash
ls -l script.sh
```

Output:

```
-rwxr-xr-x
```

Meaning:

```
Owner  : rwx
Group  : r-x
Others : r-x
```

Changing permissions:

```bash
chmod 755 script.sh
```

---

# Ownership Management

Linux files have owners and groups.

## Change owner

```bash
chown user file.txt
```

## Change group

```bash
chgrp group file.txt
```

---

# Disk and Memory Troubleshooting

## Check disk usage

```bash
df -h
```

## Check directory size

```bash
du -sh folder
```

## Check memory usage

```bash
free -m
```

---

# 2. Shell Scripting Revision 📝

Shell scripting is used to automate repetitive tasks in Linux.

During this journey, I created scripts for:

- Server health checks
- Backup automation
- Log analysis
- Package installation
- Cron automation

---

# Variables in Shell Script

Variables store data that can be reused.

Example:

```bash
name="Rohit"

echo $name
```

Output:

```
Rohit
```

---

# User Input

Taking input from users:

```bash
read name

echo "Hello $name"
```

---

# Command Line Arguments

Arguments allow passing values while executing scripts.

Example:

```bash
./script.sh Rohit
```

Access argument:

```bash
echo $1
```

---

# Conditional Statements

Used for decision making.

Example:

```bash
if [ $age -ge 18 ]
then
    echo "Adult"
else
    echo "Minor"
fi
```

---

# Loops in Shell Scripting

Loops execute commands repeatedly.

## For Loop

Example:

```bash
for i in 1 2 3 4 5
do
 echo $i
done
```

## While Loop

Example:

```bash
count=1

while [ $count -le 5 ]
do
 echo $count
 count=$((count+1))
done
```

---

# Functions

Functions help reuse code.

Example:

```bash
hello()
{
 echo "Hello DevOps"
}

hello
```

---

# Text Processing Commands

## grep

Search text patterns:

```bash
grep "ERROR" logfile.txt
```

## awk

Process columns:

```bash
awk '{print $1}' file.txt
```

## sed

Replace text:

```bash
sed 's/error/ERROR/g' file.txt
```

## sort

Sort output:

```bash
sort file.txt
```

## uniq

Remove duplicate values:

```bash
sort file.txt | uniq
```

---

# Error Handling

Used to make scripts safer.

Command:

```bash
set -euo pipefail
```

Meaning:

- set -e → Exit when command fails
- set -u → Error when using undefined variables
- pipefail → Detect pipeline failures

---

# Cron Jobs

Cron is used to schedule automated tasks.

Sysadmins use cron for:

- Backups
- Log rotation
- Monitoring
- Maintenance tasks


Edit cron:

```bash
crontab -e
```

Example:

```bash
0 3 * * * /home/ubuntu/backup.sh
```

Runs backup every day at 3 AM.

---

# 3. Networking Revision 🌐

Networking knowledge helps DevOps engineers troubleshoot servers and applications.

---

# Basic Networking Concepts

## IP Address

An IP address identifies a device on a network.

Example:

```
192.168.1.10
```

---

## DNS

DNS converts domain names into IP addresses.

Example:

```
google.com → IP address
```

---

## Ports

Ports identify services running on a machine.

Common ports:

| Service | Port |
|---|---|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| DNS | 53 |

---

# Networking Commands

## Check connectivity

```bash
ping google.com
```

## Check HTTP response

```bash
curl google.com
```

## Check listening ports

```bash
ss -tulnp
```

## DNS lookup

```bash
dig google.com
```

```bash
nslookup google.com
```

---

# 4. Git & GitHub Revision 🔀

Git is a distributed version control system used to track code changes.

---

# Basic Git Workflow

## Initialize repository

```bash
git init
```

## Check status

```bash
git status
```

## Add changes

```bash
git add .
```

## Commit changes

```bash
git commit -m "message"
```

## View history

```bash
git log --oneline
```

---

# Branching

Branches allow developers to work on features independently.

Create branch:

```bash
git branch feature-login
```

Switch branch:

```bash
git switch feature-login
```

---

# Git Merge

Merge combines changes from one branch into another.

## Fast Forward Merge

Happens when main branch has no new commits.

## Merge Commit

Happens when both branches have different commits.

---

# Git Rebase

Rebase moves commits on top of another branch.

Command:

```bash
git rebase master
```

Used for cleaner commit history.

---

# Git Stash

Stash temporarily stores unfinished changes.

Commands:

```bash
git stash
```

Restore changes:

```bash
git stash pop
```

---

# Git Cherry Pick

Cherry-pick applies a specific commit from another branch.

Example:

```bash
git cherry-pick <commit-id>
```

---

# Git Reset vs Revert

## Reset

Used mainly for local changes.

Types:

```bash
git reset --soft
git reset --mixed
git reset --hard
```

## Revert

Creates a new commit to undo changes.

```bash
git revert <commit-id>
```

Safe for shared branches.

---

# GitHub CLI Revision

GitHub CLI allows managing GitHub from the terminal.

Check version:

```bash
gh --version
```

Login:

```bash
gh auth login
```

Create repository:

```bash
gh repo create
```

Create issue:

```bash
gh issue create
```

Create Pull Request:

```bash
gh pr create
```

---

# Self Assessment

## Strong Areas:

✅ Linux fundamentals  
✅ Shell scripting basics  
✅ Git workflows  
✅ GitHub operations  

## Need More Practice:

🔄 Advanced networking  
🔄 Advanced shell scripting concepts  
🔄 Git reset and revert scenarios  

---

# Key Learning From Revision

The biggest learning from this revision day is:

"Consistent practice and explaining concepts are the best ways to improve technical confidence."

Revision helped me identify gaps, strengthen fundamentals, and understand where I need more hands-on practice.

---
