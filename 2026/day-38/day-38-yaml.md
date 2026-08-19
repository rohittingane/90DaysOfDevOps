# Day 38 – YAML Basics

## What is YAML?

YAML stands for **"YAML Ain't Markup Language"**. It is a simple, human-readable
way to write data — things like configuration, settings, and structured
information. Unlike JSON (which uses lots of `{}`, `[]`, and commas), YAML
uses **indentation (spaces)** to show structure, so it looks clean and is
easy to read, almost like plain English.

A `.yaml` or `.yml` file is basically a text file that stores data in the
form of **key-value pairs**, **lists**, and **nested objects**.

## Why is YAML used in DevOps?

Almost every major DevOps tool uses YAML to describe configuration:

- **GitHub Actions / GitLab CI / Jenkins** – pipeline files are written in YAML
- **Kubernetes** – Deployment, Service, Pod manifests are all YAML
- **Docker Compose** – `docker-compose.yml` defines multi-container apps
- **Ansible** – playbooks (automation scripts) are written in YAML

If the YAML syntax is wrong — even one wrong space or a tab — the whole
pipeline, deployment, or automation script can fail. So before writing any
real pipeline, it's important to be fully comfortable reading and writing
YAML by hand.

## How is YAML used / written?

Basic rules:

- Use **spaces only**, never tabs, for indentation
- **2 spaces** is the common standard for one indentation level
- `key: value` format — always put a space after the colon
- Strings usually don't need quotes, unless they contain special characters
  like `:` or `#`
- `true` / `false` (lowercase) are treated as booleans; `"true"` (in quotes)
  is treated as plain text (string)
- Lists can be written in two ways: block style (`-` per line) or inline
  style (`[item1, item2]`)
- Nested data is shown purely through indentation — the deeper it's indented,
  the deeper it is inside the structure

---

## Task 1 & 2: `person.yaml` — Key-Value Pairs and Lists

**Goal:** Write a YAML file describing myself using simple key-value pairs,
then add two different list formats to it.

```yaml
---
name: Rohit
role: DevOps Learner
experience_years: 0
learning: true

tools:
  - linux
  - Docker
  - AWS
  - Git
  - YAML

hobbies: [cricket, music, travelling]
```

**How I built this:**
- Started with plain key-value pairs (`name`, `role`, `experience_years`, `learning`)
- Made sure `learning` was written as lowercase `true` so it's treated as a
  real boolean, not a string
- Added `tools` as a **block-style list** — each tool on its own line with a
  `-` in front, indented under the key
- Added `hobbies` as an **inline (flow-style) list** — all items inside `[ ]`
  on the same line

**Q: What are the two ways to write a list in YAML?**

1. **Block style** — one item per line, each starting with a `-`:
   ```yaml
   tools:
     - Docker
     - Git
   ```
2. **Inline (flow) style** — all items inside square brackets, separated by commas:
   ```yaml
   hobbies: [cricket, music, travelling]
   ```

**Validation:**

![Task 1 - YAML basic structure](2026/day-38/Screenshots/task-1-yaml-basic-structure.png)

![Task 2 - YAML lists validation](2026/day-38/Screenshots/task-2-yaml-lists-validation.png)

I also hit a validation error along the way (used wrong key name / wrong
boolean case) and had to go back and fix it:

![Task 5 - person.yaml validation error and fix](2026/day-38/Screenshots/task-5-person-yaml-validation-error-fix.png)

---

## Task 3: `server.yaml` — Nested Objects

**Goal:** Describe a server and a database using nested keys (objects inside
objects), and understand what happens if a tab is used instead of spaces.

```yaml
---
server:
  name: web-server
  ip: 192.168.1.10
  port: 8080

database:
  host: localhost
  name: devops_db
  credentials:
    user: admin
    password: secret123
```

**How I built this:**
- `server` is a key that itself holds more keys (`name`, `ip`, `port`)
  nested inside it
- `database` goes even deeper — it holds `host`, `name`, and `credentials`,
  and `credentials` itself holds `user` and `password`
- Each level of nesting is shown purely by extra indentation — no braces
  or brackets needed like in JSON

**Verify: What happens when a tab is used instead of spaces?**

I intentionally replaced a space with a Tab in the indentation and ran
`yamllint` again. Instead of a clear "tab not allowed" message, it gave a
confusing-looking error:

```
4:6   error   syntax error: mapping values are not allowed here (syntax)
```

**Why this happens:** The tab breaks the indentation level YAML is expecting.
Because the structure suddenly looks wrong to the parser, it assumes a
`key: value` pair is being written in an invalid position — so the error
looks like a syntax problem, not a "you used a tab" problem. This is an
important real-world lesson: **tab-related YAML errors often don't say
"tab" anywhere in the message.**

**Validation:**

![Task 3 - Nested objects validation](2026/day-38/Screenshots/task-3-nested-objects-validation.png)

![Task 5 - server.yaml validation error and fix](2026/day-38/Screenshots/task-5-server-yaml-validation-error-fix.png)

---

## Task 4: Multi-line Strings

**Goal:** Add a `startup_script` field to `server.yaml` using both YAML
multi-line string styles: `|` (literal block) and `>` (folded block).

```yaml
startup_script_pipe: |
  #!/bin/bash
  echo "Starting server"
  echo "Server is ready"

startup_script_fold: >
  This is a startup message
  for the web server
  when it starts.
```

**How I built this:**
- Used `|` for the script — this preserves every line break exactly as
  written, which matters for shell scripts because each command needs to
  stay on its own line
- Used `>` for a plain message — this "folds" multiple lines into one single
  line (line breaks become spaces), which is fine for a plain sentence where
  line breaks were only there for readability

**Q: When would you use `|` vs `>`?**

- **`|` (pipe / literal style):** Use when line breaks are important —
  scripts, code, config files, anything where each line must stay separate.
- **`>` (fold style):** Use for plain text/paragraphs where you just want
  the text to be readable across multiple lines in the file, but it should
  be treated as one continuous piece of text (like a single sentence or
  message).

**Validation:**

![Task 4 - Multi-line strings validation](2026/day-38/Screenshots/task-4-multi-line-strings-validation.png)

---

## Task 5: Validating YAML with `yamllint`

**Goal:** Install and use `yamllint` to check both YAML files for errors,
break the indentation on purpose, see the error, then fix it.

**Steps I followed:**
1. Ran `yamllint person.yaml` and `yamllint server.yaml` on the clean files
   — no errors after fixing key names and boolean casing
2. Intentionally broke indentation (mismatched spacing between nested
   levels) → got errors like:
   ```
   wrong indentation: expected X but found Y (indentation)
   ```
3. Intentionally added a Tab character → got:
   ```
   syntax error: mapping values are not allowed here (syntax)
   ```
4. Fixed the indentation to be consistent throughout the file → `yamllint`
   ran clean with no output (no output = no errors, meaning the file is valid)

**What I understood from this:** `yamllint` doesn't require an exact number
of spaces (like "must be 2"), but it does require the indentation to be
**consistent** — the same nesting level must always use the same number of
spaces throughout the file.

---

## Task 6: Spot the Difference

```yaml
# Block 1 - correct
name: devops
tools:
  - docker
  - kubernetes
```

```yaml
# Block 2 - broken
name: devops
tools:
- docker
  - kubernetes
```

**What's wrong with Block 2:**

The two list items under `tools` are **not indented the same way**:
- `- docker` starts at column 0, at the same indentation level as `tools:` itself
- `- kubernetes` is indented 2 spaces further than `- docker`

Because of this mismatch, YAML doesn't read them as one single, consistent
list — the structure becomes ambiguous/broken, and this would cause a
parsing or indentation error. In Block 1, both list items are indented
equally under `tools:`, which is the correct way to write a block-style list.

---

## What I Learned (3 Key Points)

1. **Spaces only, never tabs.** YAML is very sensitive to whitespace, and
   using a tab doesn't throw an obvious "tab not allowed" message — instead
   it shows up as a confusing generic syntax error, which can be misleading
   if you don't already know tabs are the cause.

2. **Indentation needs to be consistent, not a fixed number.** YAML doesn't
   force you to use exactly 2 spaces — what actually matters is that every
   item at the same nesting level uses the exact same amount of indentation
   throughout the file.

3. **`|` and `>` serve different purposes for multi-line text.** `|` keeps
   every line break exactly as written (good for scripts/code), while `>`
   folds multiple lines into a single line (good for plain paragraphs or
   messages where line breaks in the source file are just for readability).
