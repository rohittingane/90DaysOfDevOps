# Day 13 – Linux Logical Volume Manager (LVM)

## 📌 Objective

Learn how Linux Logical Volume Manager (LVM) provides flexible storage management by creating, mounting, verifying, and extending Logical Volumes without repartitioning disks.

---

# Task 1 – Switch to Root User

Before performing LVM operations, switch to the root user because storage management requires administrative privileges.

## Command

```bash
sudo -i
```

## What?

Switches the current user to the root user.

## Why?

LVM commands require root privileges.

## How?

After switching, all administrative commands can be executed without using `sudo` repeatedly.

### 📸 Output

![Root User](Screenshots/01-root-user.png)

---

# Task 2 – Check Current Storage

Before creating LVM, check the existing disks, mounted filesystems, and current LVM configuration.

## Commands

```bash
lsblk
```

```bash
df -h
```

```bash
pvs
```

```bash
vgs
```

```bash
lvs
```

## What?

Displays available disks, mounted storage, and existing LVM configuration.

## Why?

To understand the current storage layout before making any changes.

## How?

These commands verify whether any Physical Volumes, Volume Groups, or Logical Volumes already exist.

### 📸 Output

![Current Storage](Screenshots/02-current-storage-check.png)

---

# Task 3 – Create an EBS Volume

Created a new EBS volume from the AWS Console.

## What?

An EBS Volume is additional block storage attached to an EC2 instance.

## Why?

LVM requires additional storage devices for creating Physical Volumes.

## How?

Created a new EBS volume in the same Availability Zone as the EC2 instance.

### 📸 Output

![EBS Volume Created](Screenshots/03-ebs-volume-created.png)

---

# Task 4 – Attach the EBS Volume

Attached the newly created EBS volume to the EC2 instance.

## What?

Attaches storage to the running EC2 instance.

## Why?

Linux can only use a disk after it is attached.

## How?

Selected the running EC2 instance and attached the EBS volume using `/dev/sdf`.

### 📸 Output

![Attach EBS Volume](Screenshots/04-attaching-ebs-volume.png)

---

# Task 5 – Verify Attached Disks

Verified that Linux detected the newly attached disks.

## Command

```bash
lsblk
```

## What?

Lists all available block devices.

## Why?

To verify that the newly attached EBS disks are visible.

## How?

The newly attached disks appeared as:

- nvme1n1
- nvme2n1
- nvme3n1

### 📸 Output

![Attached Volumes](Screenshots/05-lsblk-after-volume-attachment.png)

---

# Task 6 – Create Physical Volumes (PV)

Initialized the disks as Physical Volumes.

## Commands

```bash
sudo pvcreate /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
```

```bash
sudo pvs
```

## What?

Converts raw disks into Physical Volumes.

## Why?

LVM can only manage disks after they become Physical Volumes.

## How?

Each attached EBS volume was initialized using `pvcreate`.

### 📸 Output

![Physical Volume](Screenshots/06-physical-volume-created.png)

---

# Task 7 – Create Volume Group (VG)

Combined multiple Physical Volumes into one storage pool.

## Commands

```bash
sudo vgcreate rohit_vg /dev/nvme1n1 /dev/nvme2n1
```

```bash
sudo vgs
```

## What?

Creates a Volume Group.

## Why?

Volume Groups combine multiple disks into a single storage pool.

## How?

Combined two Physical Volumes into one Volume Group named `rohit_vg`.

### 📸 Output

![Volume Group](Screenshots/07-volume-group-created.png)

---

# Task 8 – Create Logical Volume (LV)

Created a Logical Volume inside the Volume Group.

## Commands

```bash
sudo lvcreate -L 15G -n rohit_lv rohit_vg
```

```bash
sudo lvs
```

## What?

Creates a Logical Volume.

## Why?

Logical Volumes are used by applications instead of raw disks.

## How?

Allocated 15GB from the Volume Group.

### 📸 Output

![Logical Volume](Screenshots/08-logical-volume-created.png)

---

# Task 9 – Format and Mount the Logical Volume

Created a filesystem and mounted the Logical Volume.

## Commands

```bash
sudo mkfs.ext4 /dev/rohit_vg/rohit_lv
```

```bash
sudo mkdir /mnt/rohit_lv_mount
```

```bash
sudo mount /dev/rohit_vg/rohit_lv /mnt/rohit_lv_mount
```

```bash
df -h
```

## What?

Creates an EXT4 filesystem and mounts it.

## Why?

A Logical Volume must be formatted before storing files.

## How?

Formatted using EXT4, mounted it, and verified using `df -h`.

### 📸 Output

![Logical Volume Mounted](Screenshots/09-logical-volume-mounted.png)

---

# Task 10 – Verify Mounted Logical Volume

Created sample files and directories to verify the mounted storage.

## Commands

```bash
cd /mnt/rohit_lv_mount
```

```bash
mkdir rohit_devops
```

```bash
ls
```

```bash
ls -ld rohit_devops
```

## What?

Verified the mounted filesystem.

## Why?

To ensure files can be created successfully.

## How?

Created a directory and confirmed it exists.

### 📸 Output

![Logical Volume Verification](Screenshots/10-logical-volume-verification.png)

---

# Task 11 – Extend the Logical Volume

Increased the storage capacity without recreating partitions.

## Commands

```bash
sudo lvextend -L +200M /dev/rohit_vg/rohit_lv
```

```bash
sudo resize2fs /dev/rohit_vg/rohit_lv
```

```bash
df -h /mnt/rohit_lv_mount
```

## What?

Extends the Logical Volume and resizes the filesystem.

## Why?

Applications may require additional storage as data grows.

## How?

Extended the Logical Volume by 200MB and resized the EXT4 filesystem online.

### 📸 Output

![Logical Volume Extended](Screenshots/11-logical-volume-extended-200mb.png)

---

# 📚 Commands Used

| Command | Description |
|----------|-------------|
| sudo -i | Switch to root user |
| lsblk | List block devices |
| df -h | Show mounted filesystems |
| pvs | Display Physical Volumes |
| vgs | Display Volume Groups |
| lvs | Display Logical Volumes |
| pvcreate | Create Physical Volume |
| vgcreate | Create Volume Group |
| lvcreate | Create Logical Volume |
| mkfs.ext4 | Format Logical Volume |
| mkdir | Create mount point |
| mount | Mount filesystem |
| lvextend | Extend Logical Volume |
| resize2fs | Resize EXT4 filesystem |

---

# 🎯 What I Learned

- Learned the complete LVM architecture (Physical Volume → Volume Group → Logical Volume).
- Learned how to create and manage flexible storage using LVM.
- Learned how to format and mount Logical Volumes.
- Learned how to verify storage using `lsblk`, `pvs`, `vgs`, `lvs`, and `df -h`.
- Learned how to extend storage without repartitioning disks.
- Learned how `resize2fs` expands the filesystem after increasing Logical Volume size.
- Understood why LVM is widely used in production Linux servers.

---

# ✅ Outcome

Successfully created and managed Linux Logical Volume Manager (LVM) using AWS EC2. Created Physical Volumes, Volume Groups, Logical Volumes, mounted storage, verified file operations, and dynamically extended storage without downtime.
