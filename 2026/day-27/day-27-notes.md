# Day 27 – GitHub Profile Makeover Notes

## Objective

Today I improved my GitHub profile and organized my repositories to create a professional developer identity.

---

## Changes Made

### 1. GitHub Profile README

Created a profile README repository:

Repository:
rohittingane/rohittingane

Added:

- Introduction about my DevOps journey
- Current learning focus
- Tech stack
- Featured projects
- Contact information

---

### 2. Repository Organization

Organized my DevOps projects:

### 90DaysOfDevOps
- Daily DevOps learning journey
- Linux, Git, AWS, Docker and CI/CD practice

### shell-scripts
- Collection of Bash automation scripts
- Health checks
- Backup scripts
- System automation scripts

### devops-notes
- Linux notes
- Git commands
- Shell scripting references
- DevOps documentation

---

## Security Check

Verified repositories for sensitive data:

Commands used:

```bash
find . -name ".env"
find . -name "*.pem"
grep -R "password\|secret\|token\|apikey" .
