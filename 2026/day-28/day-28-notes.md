# Day 28 – Revision Day: Everything from Day 1 to Day 27

## Overview

Day 28 was a revision day to review everything I learned during the first 27 days of my DevOps journey.

I revised:

- DevOps fundamentals
- Linux administration
- Shell scripting
- Docker and Cloud basics
- Networking concepts
- Git & GitHub workflows
- GitHub CLI
- Developer branding

The main goal was to identify my strong areas, weak areas, and improve confidence through revision and hands-on practice.

---

# Self Assessment Checklist

## Linux

### Confident

✅ Linux file system navigation  
✅ Create, move, copy, and delete files/directories  
✅ Process management using ps, top, kill  
✅ Systemd service management  
✅ File editing using nano  
✅ CPU, memory, and disk troubleshooting  
✅ Linux file system hierarchy  
✅ User and group management  
✅ File permissions using chmod  
✅ Ownership management using chown and chgrp  

### Need More Practice

🔄 LVM volume management  
🔄 Advanced networking troubleshooting  
🔄 DNS, IP addressing, and subnet concepts  

---

# Shell Scripting

### Confident

✅ Variables and user input  
✅ Command line arguments  
✅ if/else conditions  
✅ for, while, until loops  
✅ Functions  
✅ Cron jobs  
✅ Backup scripts  
✅ Log analyzer projects  

### Need More Practice

🔄 Advanced text processing using awk and sed  
🔄 Error handling using set -euo pipefail  

---

# Git & GitHub

### Practiced Concepts

✅ Initialize repository  
✅ Stage and commit changes  
✅ View commit history  
✅ Create and switch branches  
✅ Push and pull changes from GitHub  
✅ Clone vs Fork  
✅ Merge branches  
✅ Fast-forward merge  
✅ Merge commit  
✅ Rebase  
✅ Git stash and stash pop  
✅ Cherry-pick commits  
✅ Squash merge  
✅ Git reset  
✅ Git revert  
✅ GitHub CLI  

---

# Git Reset Revision

## Soft Reset

Command:

```bash
git reset --soft HEAD~1
```

Soft reset removes the commit but keeps the changes staged.

Result:

```
Commit removed
Changes staged
Files preserved
```

Used when we want to modify a previous commit or combine commits.

---

## Mixed Reset

Command:

```bash
git reset --mixed HEAD~1
```

Mixed reset removes the commit and unstages the changes.

Result:

```
Commit removed
Changes unstaged
Files preserved
```

This is the default reset mode.

---

## Hard Reset

Command:

```bash
git reset --hard HEAD~1
```

Hard reset removes the commit and deletes all changes.

Result:

```
Commit removed
Changes removed
Working tree cleaned
```

Should be used carefully because changes can be lost permanently.

---

# Git Revert Revision

Git revert is a safer way to undo changes.

Unlike reset, revert does not remove history.

It creates a new commit that reverses the previous commit.

Command:

```bash
git revert <commit-id>
```

Git revert is commonly used on shared branches like main/master because it keeps the history safe.

---

# Networking Revision

Important commands practiced:

## Check Connectivity

```bash
ping google.com
```

## Check HTTP Response

```bash
curl example.com
```

## Check Open Ports

```bash
ss -tulnp
```

## DNS Lookup

```bash
dig google.com
```

```bash
nslookup google.com
```

Common Ports:

| Service | Port |
|---|---|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| DNS | 53 |
| MySQL | 3306 |

---

# Shell Text Processing Revision

## grep

Used to search patterns in files.

Example:

```bash
grep "ERROR" logfile.txt
```

## awk

Used for processing columns and structured text.

Example:

```bash
awk '{print $1}' file.txt
```

## sed

Used for searching and replacing text.

Example:

```bash
sed 's/error/ERROR/g' file.txt
```

## sort and uniq

Used for sorting and removing duplicate values.

Example:

```bash
sort file.txt | uniq
```

---

# Quick Fire Questions

## 1. What does chmod 755 script.sh do?

755 means:

Owner:
- Read
- Write
- Execute

Group:
- Read
- Execute

Others:
- Read
- Execute

Example:

```bash
chmod 755 script.sh
```

---

## 2. Difference between Process and Service

Process:

- A running instance of a program.

Service:

- A background process managed by systemd.

Example:

Nginx running = Process

Nginx managed by systemd = Service

---

## 3. Find process using port 8080

```bash
ss -tulnp | grep 8080
```

or

```bash
lsof -i :8080
```

---

## 4. What does set -euo pipefail do?

```bash
set -euo pipefail
```

- `-e` → Exit script when a command fails
- `-u` → Error when using undefined variables
- `pipefail` → Detect failures inside pipelines

---

## 5. Difference between git reset --hard and git revert

Git reset:

- Changes history
- Mostly used for local changes

Git revert:

- Creates a new undo commit
- Safe for shared branches

---

## 6. Recommended branching strategy for a team of 5 developers

GitHub Flow is suitable for a small team.

Flow:

1. Create feature branch
2. Make changes
3. Push branch
4. Create Pull Request
5. Review and merge

---

## 7. What does git stash do?

Git stash temporarily stores uncommitted changes.

Example:

```bash
git stash
```

Restore changes:

```bash
git stash pop
```

Useful when we need to switch branches without committing unfinished work.

---

## 8. Schedule a script every day at 3 AM

Using crontab:

```bash
crontab -e
```

Add:

```bash
0 3 * * * /path/script.sh
```

---

## 9. Difference between git fetch and git pull

git fetch:

- Downloads latest changes from remote
- Does not merge automatically

git pull:

- Fetch + merge changes

---

## 10. What is LVM?

LVM (Logical Volume Manager) provides flexible disk management.

Advantages:

- Resize volumes easily
- Manage storage dynamically
- Combine multiple disks

---

# Teach It Back

## Explain Git Branching

Git branching allows developers to work on different features independently without affecting the main code.

A branch creates a separate development path where changes can be developed and tested.

Example:

```bash
git branch feature-login
git switch feature-login
```

After completing work, the branch can be merged back into the main branch.

Branching helps teams collaborate safely and manage multiple features efficiently.

---

## Explain File Permissions to a New Linux User

Linux file permissions control who can read, write, or execute a file.

There are three types of users:

- Owner
- Group
- Others

Permission types:

- r → Read
- w → Write
- x → Execute

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

File permissions help maintain security by controlling access to files and directories.

---

## Explain What a Crontab Is and Why Sysadmins Use It

Crontab is a Linux utility used to schedule commands or scripts automatically at a specific time.

Sysadmins use cron jobs for:

- Backups
- Log rotation
- System monitoring
- Maintenance tasks
- Cleaning temporary files

Edit cron jobs:

```bash
crontab -e
```

Example:

```bash
0 3 * * * /home/ubuntu/backup.sh
```

This runs the backup script every day at 3 AM.

Cron jobs help automate repetitive tasks and reduce manual effort.

---

# Day 28 Summary

Day 28 helped me revise my complete DevOps learning journey from Day 1 to Day 27.

I reviewed Linux administration, Shell scripting, Networking, Git & GitHub, and GitHub CLI.

I identified topics that need more practice and improved my understanding through hands-on revision.

Continuous practice and explaining concepts are helping me build stronger DevOps fundamentals.

---
