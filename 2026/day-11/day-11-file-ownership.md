# Day 11 – File Ownership 

## Objective

Today's goal was to understand how Linux file ownership works and learn how to manage file and directory ownership using `chown` and `chgrp`.

---

# Task 1 – Understanding File Ownership

Created a file and checked its default owner and group.

### Commands

```bash
touch devops-file.txt
ls -l devops-file.txt
```

### Output

- Owner: ubuntu
- Group: ubuntu

---

# Task 2 – Change File Owner

Changed the owner of a file using `chown`.

### Commands

```bash
sudo chown tokyo devops-file.txt
ls -l devops-file.txt

sudo chown berlin devops-file.txt
ls -l devops-file.txt
```

### Result

- Owner changed successfully.
- Group remained unchanged.

---

# Task 3 – Change File Group

Created a new group and changed the file group.

### Commands

```bash
touch team-notes.txt

sudo groupadd heist-team

grep heist-team /etc/group

sudo chgrp heist-team team-notes.txt

ls -l team-notes.txt
```

### Result

- Owner remained ubuntu.
- Group changed to heist-team.

---

# Task 4 – Change Owner and Group Together

Changed both owner and group using a single command.

### Commands

```bash
touch project-config.yaml

sudo chown professor:heist-team project-config.yaml

ls -l project-config.yaml
```

Created a directory and changed its ownership.

```bash
sudo mkdir app-logs

sudo chown berlin:heist-team app-logs

ls -ld app-logs
```

### Result

- Owner and group changed successfully.

---

# Task 5 – Recursive Ownership

Created a project directory structure and applied ownership recursively.

### Commands

```bash
mkdir -p heist-project/vault
mkdir -p heist-project/plans

touch heist-project/vault/gold.txt
touch heist-project/plans/strategy.conf

sudo groupadd planners

sudo chown -R professor:planners heist-project

ls -lR heist-project
```

### Result

Ownership of all files and subdirectories changed successfully.

---

# Task 6 – Practice Challenge

Created a practical scenario with multiple users and groups.

### Commands

```bash
sudo groupadd vault-team
sudo groupadd tech-team

mkdir bank-heist

touch bank-heist/access-codes.txt
touch bank-heist/blueprints.pdf
touch bank-heist/escape-plan.txt

sudo chown tokyo:vault-team bank-heist/access-codes.txt
sudo chown berlin:tech-team bank-heist/blueprints.pdf
sudo chown nairobi:vault-team bank-heist/escape-plan.txt

ls -l bank-heist
```

### Result

Each file has its own owner and group.

---

# Commands Practiced

- touch
- mkdir
- ls -l
- ls -ld
- ls -lR
- chown
- chgrp
- groupadd
- grep
- sudo

---

# Real Issues Faced & Solutions

## Issue 1

### Error

```
Permission denied
```

### Reason

The directory was created using `sudo mkdir`, so the owner was `root`. Since I was logged in as the `ubuntu` user, I didn't have write permission inside that directory.

### Solution

Changed the directory permission using:

```bash
sudo chmod 777 bank-heist
```

After that, I was able to create files successfully.

---

## Issue 2

### Error

```
No such file or directory
```

### Reason

I tried changing ownership from the wrong directory.

### Solution

Moved into the correct directory using:

```bash
cd bank-heist
```

Then executed the `chown` command successfully.

---

# What I Learned

- Every file has an Owner and a Group.
- `chown` changes the file owner.
- `chgrp` changes the file group.
- `chown owner:group` changes both together.
- `chown -R` changes ownership recursively.
- Directory ownership and permissions directly affect file creation.
- Always verify ownership using `ls -l`.

---

# Screenshots

- 01-understanding-file-ownership.png
- 02-change-file-owner.png
- 03-change-group-ownership.png
- 04-change-owner-and-group.png
- 05-recursive-ownership.png
- 06-multiple-file-ownership.png
