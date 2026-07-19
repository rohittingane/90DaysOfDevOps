# Git Commands Cheat Sheet

Complete list of Git commands practiced on Day 22, along with explanations for each.

---

## 1. Setup & Configuration

### Check Git Version

```bash
git --version
```

**Explanation:** Checks whether Git is installed on the system and shows the installed version. This is the first command to run after installing any tool — verify it's available before using it.

---

### Configure Username

```bash
git config --global user.name "Rohit Tingane"
```

**Explanation:** Sets the name that will be attached as the **author** of every commit. `--global` means this setting applies to all Git repositories on this machine, not just one project.

---

### Configure Email

```bash
git config --global user.email "rtingame2611@gmail.com"
```

**Explanation:** Sets the email linked to every commit. This should match the email used on GitHub, so commits are correctly associated with your GitHub account/profile.

---

### Verify Configuration

```bash
git config --list
```

**Explanation:** Displays all current Git configuration settings (username, email, and other options). Used to confirm that configuration was saved correctly — following the rule: **"Trust, but verify."**

---

## 2. Repository Management

### Create Project Directory

```bash
mkdir devops-git-practice
```

**Explanation:** `mkdir` (make directory) creates a new folder. This folder acts as the home for the project before it becomes a Git repository.

---

### Navigate to Project Directory

```bash
cd devops-git-practice
```

**Explanation:** `cd` (change directory) moves into the newly created folder so all further commands run inside the project.

---

### Initialize Git Repository

```bash
git init
```

**Explanation:** Converts a normal folder into a **Git repository** by creating a hidden `.git` directory inside it. From this point on, Git starts tracking every change made in this folder.

---

### Check Repository Status

```bash
git status
```

**Explanation:** Shows the current state of the working directory and staging area — which files are modified, staged, or untracked. Used constantly throughout a Git workflow to know what's going on before adding or committing.

---

## 3. File Management

### Create New File

```bash
touch git-commands.md
```

**Explanation:** `touch` creates a new, empty file. Used here to create the documentation file for tracking Git commands.

---

### List Files Including Hidden Files

```bash
ls -la
```

**Explanation:** Lists all files and folders in the current directory, including hidden ones (like `.git`), along with permissions, ownership, and size details.

---

### Explore Git Internal Directory

```bash
ls -la .git
```

**Explanation:** Looks inside the `.git` folder to see Git's internal structure (HEAD, config, objects, refs, etc.) — helps understand how Git stores repository data internally.

---

## 4. Viewing Information

### Check Current Working Directory

```bash
pwd
```

**Explanation:** `pwd` (print working directory) shows the full path of the folder you're currently in. Used to confirm you're working inside the correct project folder.

---

## 5. Git Workflow Commands

Git follows a three-stage workflow:

```
Working Directory  →  Staging Area  →  Repository
     (edit file)         (git add)      (git commit)
```

### Check Repository Status

```bash
git status
```

**Explanation:** Run again here in the workflow context — shows whether a file is untracked, modified, or staged, right before deciding to `add` or `commit`.

---

### Add File to Staging Area

```bash
git add git-commands.md
```

**Explanation:** Moves a file from the working directory into the **staging area** — marking it as ready to be included in the next commit. General form: `git add <filename>`.

---

### Commit Changes

```bash
git commit -m "Added Git commands documentation"
```

**Explanation:** Permanently saves the staged changes into the Git repository's history as a new **commit**, along with a descriptive message (`-m`). Each commit records the author, timestamp, and exact changes made.

---

### View Commit History

```bash
git log
```

**Explanation:** Displays the full commit history — commit ID (hash), author, date, and commit message for every commit made so far.

---

### View Short Commit History

```bash
git log --oneline
```

**Explanation:** Shows a compact, single-line version of the commit history — just the short commit hash and message. Useful for quickly scanning through many commits.

---

## Summary Flow

```
git --version
      ↓
git config --global user.name / user.email
      ↓
git config --list   (verify)
      ↓
mkdir + cd           (project folder)
      ↓
git init             (initialize repo)
      ↓
touch git-commands.md (create file)
      ↓
git status           (check state)
      ↓
git add              (stage changes)
      ↓
git commit -m ""     (save history)
      ↓
git log / git log --oneline   (view history)
```
