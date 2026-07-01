# Day 09 – Linux User & Group Management

## 📌 Objective

The goal of today's challenge was to understand Linux user and group management by performing hands-on tasks. I learned how to create users, manage groups, assign permissions, and test access in a real-world Linux environment.

---

# Task 1 – Create Users

### Completed

Created the following users with home directories and passwords:

* tokyo
* berlin
* professor

### Commands Used

```bash
sudo useradd -m tokyo
sudo passwd tokyo

sudo useradd -m berlin
sudo passwd berlin

sudo useradd -m professor
sudo passwd professor
```

### Verification

```bash
grep -E "tokyo|berlin|professor" /etc/passwd

ls -l /home
```

### What I Learned

* `useradd` creates a new Linux user.
* `-m` automatically creates the user's home directory.
* `passwd` is used to set or change a user's password.
* User details are stored in `/etc/passwd`.
* Home directories are created under `/home`.

---

# Task 2 – Create Groups

### Completed

Created two groups:

* developers
* admins

### Commands Used

```bash
sudo groupadd developers

sudo groupadd admins
```

### Verification

```bash
grep -E "developers|admins" /etc/group
```

### What I Learned

* Groups help manage permissions for multiple users.
* Group information is stored in `/etc/group`.

---

# Task 3 – Assign Users to Groups

### Completed

Assigned users as follows:

| User      | Groups             |
| --------- | ------------------ |
| tokyo     | developers         |
| berlin    | developers, admins |
| professor | admins             |

### Commands Used

```bash
sudo usermod -aG developers tokyo

sudo usermod -aG developers,admins berlin

sudo usermod -aG admins professor
```

### Verification

```bash
groups tokyo

groups berlin

groups professor
```

---

# Mistake Faced

I tried adding two users using one command:

```bash
sudo usermod -aG project-team tokyo,nairobi
```

### Error

```
usermod: user 'tokyo,nairobi' does not exist
```

### Reason

`usermod` modifies only one user at a time.

Linux treated:

```
tokyo,nairobi
```

as a single username.

### Solution

Added each user separately.

---

# Task 4 – Shared Directory

### Completed

Created a shared directory:

```
/opt/dev-project
```

Changed group ownership to:

```
developers
```

Changed permissions to:

```
775
```

Tested file creation using:

* tokyo
* berlin

### Commands Used

```bash
sudo mkdir /opt/dev-project

sudo chgrp developers /opt/dev-project

sudo chmod 775 /opt/dev-project

sudo -u tokyo touch /opt/dev-project/tokyo.txt

sudo -u berlin touch /opt/dev-project/berlin.txt
```

### Verification

```bash
ls -ld /opt/dev-project

ls -l /opt/dev-project
```

### What I Learned

* `chgrp` changes the group owner.
* `chmod 775` gives:

  * Owner → Read, Write, Execute
  * Group → Read, Write, Execute
  * Others → Read, Execute
* `sudo -u` allows testing commands as another user.

---

# Mistakes Faced

## Mistake 1

Executed:

```bash
chmod 775
```

### Error

```
missing operand
```

### Reason

I forgot to specify the target directory.

### Solution

```bash
sudo chmod 775 /opt/dev-project
```

---

## Mistake 2

Executed:

```bash
sudo chgrp developers
```

### Error

```
missing operand
```

### Reason

I specified the group name but forgot to specify the directory.

### Solution

```bash
sudo chgrp developers /opt/dev-project
```

---

# Task 5 – Team Workspace

### Completed

Created:

* User: nairobi
* Group: project-team

Added:

* tokyo
* nairobi

Created:

```
/opt/team-workspace
```

Changed group ownership and permissions.

Tested file creation using:

```
sudo -u nairobi
```

### Commands Used

```bash
sudo useradd -m nairobi

sudo passwd nairobi

sudo groupadd project-team

sudo usermod -aG project-team tokyo

sudo usermod -aG project-team nairobi

sudo mkdir /opt/team-workspace

sudo chgrp project-team /opt/team-workspace

sudo chmod 775 /opt/team-workspace

sudo -u nairobi touch /opt/team-workspace/nairobi.txt
```

### Verification

```bash
groups tokyo

groups nairobi

ls -ld /opt/team-workspace

ls -l /opt/team-workspace
```

---

# Commands Practiced

* useradd
* passwd
* groupadd
* usermod
* groups
* mkdir
* chgrp
* chmod
* touch
* grep
* ls
* sudo -u

---

# Key Learnings

* Difference between users and groups.
* Primary group vs secondary groups.
* How to assign users to multiple groups.
* Difference between Owner, Group, and Others.
* Understanding Linux permission order:

  * Owner
  * Group
  * Others
* How to test permissions using `sudo -u`.
* How shared directories work in Linux.

---

# Screenshots

* User Creation
* User Verification
* Group Creation
* Group Assignment
* Shared Directory Permissions
* File Creation Test
* Team Workspace Setup
