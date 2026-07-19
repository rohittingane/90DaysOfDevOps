# Day 23 - Git Branching & Working with GitHub

## Task 1 - Understanding Branches

### 1. What is a branch in Git?

A branch is an independent line of development in Git. It allows developers to work on new features, bug fixes, or experiments without affecting the main branch.

**Example:**

```bash
git branch feature-login
git switch feature-login
```

This creates and switches to a new branch called `feature-login`.

---

### 2. Why do we use branches instead of committing everything to main?

Branches keep the main branch stable while allowing multiple developers to work on different features independently. This makes collaboration safer and easier.

**Example:**

```text
main
 ├── Stable production code
 └── feature-login
      └── New login feature under development
```

The login feature is developed separately and merged into `main` only after testing.

---

### 3. What is HEAD in Git?

HEAD is a pointer that refers to the currently checked-out branch or commit.

**Example:**

```bash
git branch
```

Output:

```text
* feature-1
  main
```

The `*` indicates that **HEAD** is currently pointing to the `feature-1` branch.

---

### 4. What happens to your files when you switch branches?

Git updates the working directory to match the selected branch. Files unique to that branch appear, while files not present disappear or revert to that branch's version.

**Example:**

```bash
git switch feature-1
touch feature.txt
git add .
git commit -m "Add feature file"

git switch main
```

After switching back to `main`, `feature.txt` will not be visible because it exists only in the `feature-1` branch.

---

## Task 3

### Difference between origin and upstream

- **origin** is your own GitHub repository (your fork or repository).
- **upstream** is the original repository from which you forked.

**Example:**

```bash
git remote -v
```

Output:

```text
origin    https://github.com/rohittingane/devops-git-practice.git
upstream  https://github.com/TrainWithShubham/90DaysOfDevOps.git
```

---

## Task 4

### Difference between git fetch and git pull

- **git fetch** downloads the latest changes but does not merge them.
- **git pull** downloads and immediately merges the latest changes into the current branch.

**Example:**

```bash
git fetch origin
```

Downloads updates only.

```bash
git pull origin main
```

Downloads and merges the latest changes from the `main` branch.

---

## Task 5

### Difference between clone and fork

- **Clone** copies a repository from GitHub to your local machine.
- **Fork** creates your own copy of someone else's repository on GitHub.

**Example:**

Clone:

```bash
git clone https://github.com/TrainWithShubham/90DaysOfDevOps.git
```

Fork:

```text
GitHub → Click Fork → Clone your fork
```

---

### When would you clone vs fork?

- Use **Clone** when working on your own repository or when you only need a local copy.
- Use **Fork** when you want to contribute to someone else's repository without affecting the original project.

**Example:**

```text
Own Project        → Clone
Open Source Project → Fork
```

---

### How do you keep your fork in sync?

1. Add the original repository as `upstream`.
2. Fetch the latest changes.
3. Merge or rebase the updates.
4. Push the updated branch to your GitHub fork.

**Example:**

```bash
git remote add upstream https://github.com/TrainWithShubham/90DaysOfDevOps.git

git fetch upstream

git merge upstream/main

git push origin main
```
