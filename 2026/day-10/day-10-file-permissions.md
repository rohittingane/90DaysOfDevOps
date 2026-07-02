# Day 10 – File Permissions & File Operations Challenge

## 📌 Objective

The goal of this challenge was to understand Linux file operations and file permissions by performing hands-on practice.

---

# 📂 Files Created

### 1. Created an empty file

```bash
touch devops.txt
```

### 2. Created a file with content

```bash
echo "Hello i am rohit" > notes.txt
```

### 3. Created a shell script

```bash
nano script.sh
```

Added the following content:

```bash
echo "Hello DevOps"
```

---

# 📖 Reading Files

Read the content of notes.txt

```bash
cat notes.txt
```

Viewed the first 5 lines of `/etc/passwd`

```bash
head -n 5 /etc/passwd
```

Viewed the last 5 lines of `/etc/passwd`

```bash
tail -n 5 /etc/passwd
```

---

# 🔐 Understanding Linux File Permissions

Checked the default permissions of the created files.

```bash
ls -l devops.txt notes.txt script.sh
```

Default permissions:

```
-rw-rw-r--
-rw-rw-r--
-rw-rw-r--
```

Understanding permission format:

```
rwxrwxrwx

Owner | Group | Others
```

Permission values:

| Permission | Value |
|------------|-------|
| Read | 4 |
| Write | 2 |
| Execute | 1 |

---

# ✏️ Modifying Permissions

## Made the shell script executable

```bash
chmod +x script.sh
```

Executed the script

```bash
./script.sh
```

Output

```
Hello DevOps
```

---

## Set devops.txt to Read Only

```bash
chmod 444 devops.txt
```

Verified

```bash
ls -l devops.txt
```

Output

```
-r--r--r--
```

---

## Set notes.txt permission to 640

```bash
chmod 640 notes.txt
```

Verified

```bash
ls -l notes.txt
```

Output

```
-rw-r-----
```

Meaning

- Owner → Read & Write
- Group → Read Only
- Others → No Permission

---

## Created a Project Directory

```bash
mkdir project
```

Changed permission

```bash
chmod 755 project
```

Verified

```bash
ls -ld project
```

Output

```
drwxr-xr-x
```

Meaning

- Owner → Read, Write, Execute
- Group → Read & Execute
- Others → Read & Execute

---

# ⚠️ Mistakes Faced During Practice

## Mistake 1

Used

```bash
chmod +X script.sh
```

Expected

The script should become executable.

Result

```
Permission denied
```

### Reason

`+X` (capital X) does not always add execute permission.

### Solution

Used

```bash
chmod +x script.sh
```

---

## Mistake 2

Executed

```bash
./ script.sh
```

Output

```
./: Is a directory
```

### Reason

A space was added after `./`.

Linux treated `./` as the current directory instead of executing the script.

### Solution

Used

```bash
./script.sh
```

---

## Mistake 3

Tried writing to a read-only file

```bash
echo "Testing Permission" >> devops.txt
```

Output

```
Permission denied
```

### Reason

The file had only read permission.

```
-r--r--r--
```

No user had write permission.

### Solution

Add write permission when modification is required.

Example

```bash
chmod +w devops.txt
```

---

## Mistake 4

Removed execute permission

```bash
chmod -x script.sh
```

Then executed

```bash
./script.sh
```

Output

```
Permission denied
```

### Reason

The execute permission had been removed.

### Solution

```bash
chmod +x script.sh
```

---

# 📚 Commands Used

```bash
touch
echo
cat
nano
head
tail
ls -l
ls -ld
chmod
mkdir
```

---

# 🎯 Key Learnings

- Learned how Linux file permissions work.
- Understood the difference between Read, Write and Execute permissions.
- Learned symbolic and numeric permission formats.
- Practiced using chmod to modify file and directory permissions.
- Understood why "Permission denied" occurs.
- Learned the importance of execute permission for shell scripts.
- Learned the difference between file permissions and directory permissions.
- Fixed real-world permission issues through hands-on practice.

---

# 📸 Screenshots

- Create Files
- Read Files
- Initial Permissions
- Permission Modifications
- Permission Testing & Errors

---

## ✅ Challenge Status

Day 10 Challenge Completed Successfully 🎉
