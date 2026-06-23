# Day 07 - Linux File System Hierarchy & Scenario-Based Practice 🚀

## Part 1: Linux File System Hierarchy

### / (Root)

* Starting point of the Linux filesystem.
* Everything in Linux begins from `/`.

**Example**

```bash
ls -l /
```

**I would use this when:** navigating the Linux filesystem.

---

### /home

* Contains home directories of normal users.
* Stores user files and settings.

**Example**

```bash
ls -l /home
```

**I would use this when:** managing user files.

---

### /root

* Home directory of the root user.
* Used for administrative tasks.

**Example**

```bash
ls -l /root
```

**I would use this when:** working as the root user.

---

### /etc

* Stores configuration files.
* Contains settings for services like SSH, Docker, Nginx, etc.

**Example**

```bash
ls -l /etc
cat /etc/hostname
```

**I would use this when:** troubleshooting configuration issues.

---

### /var/log

* Stores system and application logs.
* Important directory for DevOps troubleshooting.

**Example**

```bash
ls -l /var/log
```

**I would use this when:** checking logs and investigating failures.

---

### /tmp

* Stores temporary files.
* Files may be deleted after reboot.

**Example**

```bash
ls -l /tmp
```

**I would use this when:** storing temporary data.

---

### /bin

* Contains essential system commands.

**Example**

```bash
ls -l /bin
```

**I would use this when:** running basic Linux commands.

---

### /usr/bin

* Contains user command binaries.

**Example**

```bash
ls -l /usr/bin
```

**I would use this when:** executing installed commands.

---

### /opt

* Contains third-party applications.

**Example**

```bash
ls -l /opt
```

**I would use this when:** checking optional software installations.

---

## Hands-on Practice

### Find Largest Log Files

```bash
du -sh /var/log/* 2>/dev/null | sort -h | tail -5
```

### View Hostname

```bash
cat /etc/hostname
```

### Check Home Directory

```bash
ls -la ~
```

---

# Part 2: Scenario-Based Practice

---

# Scenario 1: Service Not Starting

### Step 1

```bash
systemctl status ssh
```

**Why:** Verify whether the service is running or failed.

---

### Step 2

```bash
journalctl -u ssh -n 50
```

**Why:** Check recent logs to understand the issue.

---

### Step 3

```bash
journalctl -u ssh -f
```

**Why:** Monitor logs in real time.

---

### Step 4

```bash
cat /etc/ssh/sshd_config
```

**Why:** Verify service configuration.

---

## Learning

Always check status first, then logs, and finally configuration files.

---

# Scenario 2: High CPU Usage

### Step 1

```bash
top
```

**Why:** View live CPU and memory usage.

---

### Step 2

```bash
ps aux --sort=-%cpu | head -10
```

**Why:** Find top CPU-consuming processes.

---

## Learning

Identify processes consuming high resources before troubleshooting applications.

---

# Scenario 3: Finding Service Logs

### Step 1

```bash
systemctl status ssh
```

**Why:** Confirm service health.

---

### Step 2

```bash
journalctl -u ssh -n 50
```

**Why:** View recent logs.

---

### Step 3

```bash
journalctl -u ssh -f
```

**Why:** Monitor logs continuously.

---

## Learning

systemd-managed services store logs in journald.

---

# Scenario 4: File Permission Issue

### Step 1

```bash
./backup.sh
```

Output:

```text
Permission denied
```

---

### Step 2

```bash
ls -l backup.sh
```

Output:

```text
-rw-rw-r--
```

No execute permission.

---

### Step 3

```bash
chmod +x backup.sh
```

Add execute permission.

---

### Step 4

```bash
./backup.sh
```

Output:

```text
Backup Completed
```

---

## Learning

Files require execute permission (`x`) to run.

---

# Key Takeaway

DevOps is not about memorizing commands.

The troubleshooting approach should always be:

**Problem → Investigation → Logs → Root Cause → Fix**

---

## Environment

* AWS EC2
* Ubuntu Linux
* SSH
* systemd
* journalctl


# Screenshots

## File System Hierarchy
![File System Hierarchy](images/filesystem.png)

## Service Status Check
![SSH Status](images/ssh-status.png)

## SSH Logs
![Journal Logs](images/journalctl-ssh.png)

## High CPU Troubleshooting
![CPU Check](images/top-ps.png)

## File Permission Fix
![Backup Script](images/backup-permission.png`


## Day 07 Completed ✅

#90DaysOfDevOps #Linux #DevOps #AWS
