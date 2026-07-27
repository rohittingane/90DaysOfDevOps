# Linu  Hands-on Practice

## 1. File System Navigation

### Commands Practiced

...

---

# 2. Users and Groups Management

## Create Users

Commands:

```bash
sudo useradd ABC
sudo useradd XYZ
```

## Create Groups

```bash
sudo groupadd devops-2
sudo groupadd devops-3
```

## Verify Groups

```bash
cat /etc/group
```

### Mistake Learned ❌

Command tried:

```bash
cat /etc/group ABC
```

Output:

```
cat: ABC: No such file or directory
```

Reason:
`cat` file read karte. User/group check karayla `/etc/group` file already enough aahe.

Correct:

```bash
cat /etc/group
```

---

# 3. File Permissions (chmod)

Create file:

```bash
touch devops11.txt
```

Check permission:

```bash
ls -l devops11.txt
```

Output:

```
-rw-rw-r-- ubuntu ubuntu devops11.txt
```


## Numeric Permissions Practice

```bash
chmod 764 devops11.txt
```

Result:

```
-rwxrw-r--
```


```bash
chmod 700 devops11.txt
```

Result:

```
-rwx------
```


Permission values:

| Permission | Value |
|---|---|
| Read | 4 |
| Write | 2 |
| Execute | 1 |


---

# 4. Ownership Management

Change owner:

```bash
sudo chown ABC devops11.txt
```


Change group:

```bash
sudo chgrp devops-2 devops11.txt
```


Change owner and group together:

```bash
sudo chown XYZ:devops-3 rohit.txt
```


### Mistake Learned ❌

Tried:

```bash
ls -l devops11
```

Output:

```
No such file or directory
```


Reason:
Actual filename was:

```
devops11.txt
```

Correct:

```bash
ls -l devops11.txt
```

---

# 5. LVM (Logical Volume Management)


## LVM Structure

```
Disk
 |
PV
 |
VG
 |
LV
 |
Filesystem
 |
Mount Point
```


## Check Disks

```bash
lsblk
```


## Check Physical Volume

```bash
sudo pvs
```


Example:

```
/dev/nvme1n1 rohit_vg
/dev/nvme2n1 rohit_vg
```


## Check Volume Group

```bash
sudo vgs
```


Output:

```
rohit_vg  24.99G
```


## Check Logical Volume

```bash
sudo lvs
```


Output:

```
rohit_lv
```


---

# LVM Extension Practice


Initial size:

```
20GB
```


Try extend:

```bash
sudo lvextend -L +5G /dev/rohit_vg/rohit_lv
```


Error:

```
Insufficient free space
```


Reason:

Available space was less than 5GB.


Correct:

```bash
sudo lvextend -L +4G /dev/rohit_vg/rohit_lv
```


Resize filesystem:

```bash
sudo resize2fs /dev/rohit_vg/rohit_lv
```


Verify:

```bash
df -h /mnt/rohit_lv_mount
```


Final:

```
20G --> 24G
```


---

# Persistent Mount using fstab


Get UUID:

```bash
sudo blkid /dev/rohit_vg/rohit_lv
```


Backup:

```bash
sudo cp /etc/fstab /etc/fstab.backup
```


Edit:

```bash
sudo nano /etc/fstab
```


Add:

```
UUID=<uuid> /mnt/rohit_lv_mount ext4 defaults 0 2
```


Test:

```bash
sudo mount -a
```


Verify:

```bash
df -h /mnt/rohit_lv_mount
```


---

# Network Troubleshooting


## Ping

Purpose:
Check connectivity.

Example:

```bash
ping google.com
```


Working:

```
64 bytes from google.com
```


---

## Curl

Purpose:
Check HTTP response.


```bash
curl google.com
```


Output:

```
301 Moved
```


Meaning:

Server reachable but redirected.


---

# Netstat

```bash
netstat -tulnp
```


Shows:

- Listening ports
- TCP/UDP connections


---

# SS Command

Modern replacement of netstat.


```bash
sudo ss -tulnp
```


Important Ports:

```
22  SSH
80  HTTP
53  DNS
```


---

# DNS Commands


## dig

```bash
dig google.com
```


Shows:

- DNS server
- IP address
- Query response


## nslookup

```bash
nslookup google.com
```


Shows:

- Domain
- IP address


---

# Mistakes Learned During Practice


## 1. Wrong file name

Mistake:

```bash
ls -l devops11
```

Correct:

```bash
ls -l devops11.txt
```


## 2. Wrong cat usage

Mistake:

```bash
cat /etc/group ABC
```

Correct:

```bash
cat /etc/group
```


## 3. LVM extension size issue

Mistake:

```bash
lvextend -L +5G
```

Reason:

Not enough free space.

Solution:

Used:

```bash
lvextend -L +4G
```


## 4. Filesystem resize missed

After LV extension:

Only LV size increases.

Need:

```bash
resize2fs
```

to increase filesystem size.


---

# Final Verification Commands


```bash
lsblk

sudo pvs

sudo vgs

sudo lvs

df -h
```

# Linux Administration Practice Completed ✅

Topics Completed:

✅ File Management  
✅ Process Management  
✅ Systemd Services  
✅ File Permissions  
✅ Users and Groups  
✅ Ownership Management  
✅ LVM  
✅ Disk Expansion  
✅ Persistent Mount  
✅ Network Commands  
✅ DNS Troubleshooting
