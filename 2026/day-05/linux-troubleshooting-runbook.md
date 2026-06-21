# Day 05 – Linux Troubleshooting Runbook

## Target Service

SSH Service (ssh)

---

## Environment Basics

### 1. Check Kernel and System Information

```bash
uname -a
```

**Observation:** Verified kernel and system information.

### 2. Check OS Details

```bash
cat /etc/os-release
```

**Observation:** Confirmed Ubuntu operating system details.

---

## Filesystem Sanity

### 3. Create Temporary Directory

```bash
mkdir /tmp/runbook-demo
```

**Observation:** Directory was created successfully.

### 4. Copy and Verify File

```bash
cp /etc/hosts /tmp/runbook-demo/hosts-copy
ls -l /tmp/runbook-demo
```

**Observation:** File creation and copy operations worked correctly. Filesystem appears healthy.

---

## CPU & Memory Snapshot

### 5. Check Running Processes

```bash
top
```

**Observation:** System processes were running normally.

### 6. Check Memory Usage

```bash
free -h
```

**Observation:** Sufficient memory was available and no memory pressure was observed.

---

## Disk & Storage

### 7. Check Disk Usage

```bash
df -h
```

**Observation:** Root filesystem usage was around 74%, with enough free space available.

### 8. Check Log Directory Size

```bash
du -sh /var/log
```

**Observation:** Log directory size was normal. Some directories required elevated privileges.

---

## Network Snapshot

### 9. Check Listening Ports

```bash
ss -tulnp
```

**Observation:** SSH service was listening on port 22.

### 10. Verify Internet Connectivity

```bash
curl -I https://google.com
```

**Observation:** Internet connectivity was working correctly.

---

## Logs Reviewed

### 11. Review SSH Logs

```bash
journalctl -u ssh -n 50
```

**Observation:** Multiple connection attempts from unknown IP addresses were observed. Successful login activity for the ubuntu user was also recorded.

### 12. Review System Logs

```bash
tail -n 50 /var/log/syslog
```

**Observation:** No critical errors were found.

---

## Quick Findings

* CPU and memory usage were normal.
* Disk space was available.
* SSH service was running properly.
* Port 22 was listening.
* SSH logs showed normal activity and some external connection attempts.
* No critical issues were observed.

---

## If This Worsens

1. Restart the SSH service if necessary.

```bash
sudo systemctl restart ssh
```

2. Review detailed logs and enable additional logging if required.

```bash
journalctl -u ssh -xe
```

3. Investigate resource usage and collect more diagnostic information.

```bash
top
free -h
df -h
```
