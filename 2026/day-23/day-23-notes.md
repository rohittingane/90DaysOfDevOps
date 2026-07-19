# Day 23 - Git Branching & Working with GitHub

## Task 1 - Understanding Branches

### 1. What is a branch in Git?

A branch is an independent line of development in Git. It allows developers to work on new features, bug fixes, or experiments without affecting the main branch.

**Example:**

```bash
git branch feature-login
git switch feature-login
```

This creates a new branch named `feature-login` and switches to it.

---

### 2. Why do we use branches instead of committing everything to main?

Branches keep the `main` branch stable while allowing developers to work on new features, bug fixes, or experiments independently. Once the work is tested, it can be merged into the `main` branch.

**Example:**

```text
main
 ├── Stable Code
 └── feature-login
      └── Login Feature Development
```

The login feature is developed separately without affecting the stable code in `main`.

---

### 3. What is HEAD in Git?

HEAD is a pointer that refers to the currently checked-out branch or commit. It tells Git where you are currently working.

**Example:**

```bash
git branch
```

Output:

```text
* feature-1
  main
```

The `*` symbol indicates that **HEAD** is currently pointing to the `feature-1` branch.

---

### 4. What happens to your files when you switch branches?

When you switch branches, Git updates your working directory to match the selected branch. Files that belong only to another branch will disappear, and files in the selected branch will appear.

**Example:**

```bash
git switch feature-1
echo "Feature work" > feature.txt
git add .
git commit -m "Add feature.txt"

git switch main
ls
```

After switching back to the `main` branch, `feature.txt` will not be visible because it exists only in the `feature-1` branch.
