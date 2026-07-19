# 📅 Day 22 — Git Fundamentals & Version Control

## 📌 Overview

Day 22 of my DevOps learning journey focused on **Git**, the foundation for GitHub, CI/CD, Jenkins, and GitHub Actions. Without a solid grip on Git, none of the higher-level DevOps tools make sense.

This document covers Git installation, configuration, repository creation, staging, committing, and building a commit history — with real-world DevOps context.

---

## ✅ Tasks Completed

| Task | Description | Status |
|------|--------------|--------|
| 1 | Install & Configure Git | ✅ |
| 2 | Create First Git Repository | ✅ |
| 3 | Create `git-commands.md` Documentation | ✅ |
| 4 | Stage & Commit Changes | ✅ |
| 5 | Build Commit History | ✅ |
| 6 | Theory Notes (`day-22-notes.md`) | ✅ |

---

## 1. What is Git?

Git is a **Distributed Version Control System (DVCS)** that tracks every change made to a project.

Think of it like a save-game system — every save point can be revisited, compared, or restored.

```
Working Code
     ↓
   Git
     ↓
Commit 1 → Commit 2 → Commit 3
     ↓
Complete History
```

## Why Git?

| Without Git | With Git |
|---|---|
| Accidental deletion = code lost forever | Restore any previous version in seconds |
| No record of who changed what | Every commit stores author, email, timestamp |
| Risky collaboration between developers | Clean, trackable collaboration history |

**Real DevOps Scenario:**

```
Developer writes Version 1
        ↓
   Pushes to Git
        ↓
Adds new feature → Bug introduced → Production breaks
        ↓
   git checkout <previous_commit>
        ↓
   Application working again ✅
```

---

## 2. Task 1 — Install & Configure Git

### Commands Used

```bash
# Check Git version
git --version

# Configure global username
git config --global user.name "Rohit Tingane"

# Configure global email (same as GitHub account)
git config --global user.email "your-email@example.com"

# Verify configuration
git config --list
```

### Why These Commands Matter

- `git --version` → confirms Git is installed correctly.
- `git config --global user.name` → sets the identity attached to every commit (**global** = applies across all repositories on the machine).
- `git config --global user.email` → links commits to your GitHub account.
- `git config --list` → verifies configuration (DevOps golden rule: **"Trust, but verify."**)

### Expected Output

```
git version 2.43.0
user.name=Rohit Tingane
user.email=your-email@example.com
```

📸 **Screenshot:** `screenshots/01-git-install-config.png`

---

## 3. Task 2 — Create First Git Repository

A **repository** is a project folder that Git actively tracks — storing source code, history, commits, and branches.

```
Repository = Project Folder + Git Memory
```

### Commands Used

```bash
# Create project folder
mkdir devops-git-practice

# Move into the folder
cd devops-git-practice

# Convert the folder into a Git repository
git init

# Verify repository creation
ls -la

# Check repository status
git status
```

### Real DevOps Example

```
payment-service/   (no Git)   → changes untracked, no accountability
payment-service/   (with Git) → every developer's change is tracked
```

📸 **Screenshot:** `screenshots/02-create-git-repository.png`

---

## 4. Task 3 — Git Commands Documentation

Created a `git-commands.md` cheat sheet inside the repo to document every command learned — because in real teams, **documentation is as important as code**.

### Commands Used

```bash
# Create the documentation file
touch git-commands.md

# Verify file creation
ls -la
```

### git-commands.md Contents

```markdown
# Git Commands Cheat Sheet

## 1. Setup & Configuration
git --version
git config --global user.name "Rohit Tingane"
git config --global user.email "rtingame2611@gmail.com"
git config --list

## 2. Repository Management
mkdir devops-git-practice
cd devops-git-practice
git init
git status

## 3. File Management
touch git-commands.md
ls -la
ls -la .git

## 4. Viewing Information
pwd

## 5. Git Workflow Commands
git status
git add filename
git commit -m "commit message"
git log
git log --oneline
```

📸 **Screenshots:**
- `Screenshots/03.1-git-commands-documentation-part1.png`
- `Screenshots/03.2-git-commands-documentation-part2.png`
- `Screenshots/03.3-git-commands-documentation-part3.png`

---

## 5. Task 4 — Stage & Commit

Git works in **three stages**:

```
Working Directory  →  Staging Area  →  Repository
   (git add)          (git commit)
```

| Area | Description | Command |
|---|---|---|
| Working Directory | Where files are created/edited | — |
| Staging Area | Changes marked ready to commit | `git add filename` |
| Repository | Permanent commit history | `git commit -m "message"` |

### Commands Used

```bash
git status
git add git-commands.md
git commit -m "Added Git commands documentation"
git log
```

📸 **Screenshot:** `Screenshots/04-git-stage-commit-history.png`

---

## 6. Task 5 — Build Commit History

Made further edits to `git-commands.md` and created a second commit to demonstrate an evolving commit history.

### Commands Used

```bash
git add git-commands.md
git commit -m "Updated git-commands.md with workflow commands"
git log --oneline
```

📸 **Screenshot:** `Screenshots/05-build-commit-history.png`

---

## 7. Task 6 — Theory Notes (`day-22-notes.md`)

A full theory write-up covering Git concepts, workflow, and interview prep, saved as `day-22-notes.md` in the repository.

📸 **Screenshot:** `Screenshots/06-day-22-notes.png`

---

## 8. Real DevOps Pipeline Context

```
Developer
    ↓
Code Changes
    ↓
Git Commit
    ↓
GitHub Repository
    ↓
CI/CD Pipeline
    ↓
Deployment
```

If production breaks, engineers use Git history to roll back to the last known working commit.

---

## 9. Interview Questions & Answers

**Q1. What is Git?**
> Git is a distributed version control system used to track changes in source code, maintain history, and support collaboration between developers.

**Q2. Why do we configure `user.name` and `user.email` in Git?**
> Git uses this information to identify who created each commit. Every commit stores the author's name and email, making it easy to track changes and maintain accountability.

**Q3. Difference between `git add` and `git commit`?**
> `git add` moves changes into the staging area; `git commit` permanently saves staged changes into the Git history.

**Q4. Why do we use `git log`?**
> `git log` displays the complete commit history, including commit ID, author, date, and commit message.

---

## 📂 Repository Structure

```
devops-git-practice/
├── .git/
├── git-commands.md
├── day-22-notes.md
└── screenshots/
    ├── 01-git-install-config.png
    ├── 02-create-git-repository.png
    ├── 03.1-git-commands-documentation-part1.png
    ├── 03.2-git-commands-documentation-part2.png
    ├── 03.3-git-commands-documentation-part3.png
    ├── 04-git-stage-commit-history.png
    ├── 05-build-commit-history.png
    └── 06-day-22-notes.png
```

---

## 🔑 Key Learning

Git is the foundation of modern DevOps workflows. A solid understanding of Git is essential before moving on to GitHub, CI/CD pipelines, Jenkins, and deployment automation.

---

## ✅ Day 22 Summary

- [x] Installed and configured Git
- [x] Created and verified a Git repository
- [x] Documented Git commands (`git-commands.md`)
- [x] Practiced staging and committing changes
- [x] Built a multi-commit history
- [x] Wrote full theory notes (`day-22-notes.md`)

**Author:** Rohit Tingane
