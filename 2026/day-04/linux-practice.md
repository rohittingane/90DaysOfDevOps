# Linux Practice - Day 04

## Process Checks

### 1. List all running processes

Command:

```bash
ps aux
```

Purpose:
View all running processes.

### 2. Monitor processes in real time

Command:

```bash
top
```

Purpose:
Monitor CPU and memory usage.

### 3. Find SSH process

Command:

```bash
pgrep ssh
```

Purpose:
Verify that SSH processes are running.

---

## Service Checks

### 1. Check SSH service status

Command:

```bash
systemctl status ssh
```

Purpose:
Verify that the SSH service is active and running.

### 2. List active services

Command:

```bash
systemctl list-units --type=service
```

Purpose:
Display all active services.

---

## Log Checks

### 1. View recent SSH logs

Command:

```bash
journalctl -u ssh --no-pager | tail -20
```

Purpose:
Display the latest SSH log entries.

### 2. View recent system logs

Command:

```bash
journalctl -xe --no-pager | tail -20
```

Purpose:
Inspect recent system events and errors.

---

## Mini Troubleshooting Steps

Problem:
Check whether the SSH service is running properly.

Step 1: Check service status

```bash
systemctl status ssh
```

Result:
SSH service is active and running.

Step 2: Verify SSH process

```bash
pgrep ssh
```

Result:
SSH processes are present.

Step 3: Inspect SSH logs

```bash
journalctl -u ssh --no-pager | tail -10
```

Result:
Recent logs show normal SSH activity.

Conclusion:
SSH service is running correctly and no critical errors were observed.
