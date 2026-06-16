# Linux Architecture, Processes and systemd

## Linux Architecture

Linux operating system is mainly divided into three parts:

### 1. Kernel
- Kernel is the core of Linux.
- It manages CPU, memory, storage, and hardware devices.
- It acts as a bridge between hardware and software.

### 2. User Space
- This is where users and applications work.
- Commands like ls, pwd, ps, and top run in user space.
- User space communicates with the kernel through system calls.

### 3. systemd (Init System)
- systemd is the first process started after the Linux kernel boots.
- It usually runs with PID 1.
- It manages services, processes, and system startup.

---

## Process Management

A process is a running instance of a program.

Examples:
- Running a command
- Starting a service
- Executing a shell script

### Process States

| State | Description |
|---------|-------------|
| Running (R) | Process is actively using CPU |
| Sleeping (S) | Waiting for an event or resource |
| Stopped (T) | Process has been paused |
| Zombie (Z) | Process has finished but still exists in the process table |

---

## Why systemd is Important

- Starts services during system boot.
- Stops and restarts services.
- Keeps track of service status.
- Helps in troubleshooting through logs.

Common Commands:

```bash
systemctl status ssh
systemctl start ssh
systemctl stop ssh
systemctl restart ssh
```

## Commands I Practiced Today

```bash
ps aux
top
systemctl status ssh
journalctl -n 20
kill
```

### Purpose of These Commands

- ps aux → View all running processes.
- top → Monitor CPU, memory, and processes in real time.
- systemctl status ssh → Check service status.
- journalctl -n 20 → View recent system logs.
- kill → Terminate a process using its PID.

---

## My Learning

Today I learned how Linux manages processes and services. I also practiced process monitoring, service management, log analysis, and process termination on my Ubuntu server. Understanding these concepts will help me troubleshoot Linux systems more effectively in the future.
