# Linux Commands Cheat Sheet

## Process Management

| Command | Usage |
|----------|----------|
| ps aux | List all running processes |
| top | Monitor CPU, memory and processes in real time |
| htop | Interactive process viewer |
| kill PID | Stop a process |
| kill -9 PID | Force kill a process |
| uptime | Show system uptime and load average |
| free -m | Display memory usage |
| pgrep process_name | Find process ID by name |

---

## File System

| Command | Usage |
|----------|----------|
| pwd | Show current directory |
| ls -l | List files with details |
| cd directory | Change directory |
| mkdir dirname | Create a directory |
| touch file.txt | Create an empty file |
| cp source destination | Copy files |
| mv source destination | Move or rename files |
| rm file.txt | Delete a file |
| df -h | Show disk usage |
| du -sh * | Show directory sizes |

---

## Networking Troubleshooting

| Command | Usage |
|----------|----------|
| ip addr | Show IP addresses |
| ping google.com | Test network connectivity |
| curl https://google.com | Test HTTP response |
| dig google.com | DNS lookup |
| ss -tulnp | Show listening ports |
| netstat -tulnp | Display network connections |

---

## Services and Logs

| Command | Usage |
|----------|----------|
| systemctl status ssh | Check SSH service status |
| systemctl start service | Start a service |
| systemctl stop service | Stop a service |
| systemctl restart service | Restart a service |
| journalctl -xe | View system logs |
| journalctl -u ssh | View SSH logs |

---

## Commands Practiced on EC2

```bash
uptime
free -m
df -h
top
ip addr
ping -c 4 google.com
systemctl status ssh
journalctl -u ssh --no-pager | tail -10
```

These commands were practiced on an Ubuntu EC2 instance as part of Day 03 of the 90 Days of DevOps challenge.
