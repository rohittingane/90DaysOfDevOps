# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

## 📖 Overview

Today I continued learning Shell Scripting by exploring loops, command-line arguments, package automation, and error handling. These concepts are widely used in DevOps to automate repetitive tasks, make scripts reusable, and handle failures safely.

---

# Task 1 – For Loop

## Scripts
- `for_loop.sh`
- `count.sh`

### Purpose
Learned how to use a **for loop** to execute the same task multiple times without repeating code.

### What I Practiced
- Printed a list of fruits using a for loop.
- Printed numbers from 1 to 10 using a for loop.

### Concepts Learned
- for loop
- Iteration
- Variables inside loops

---

# Task 2 – While Loop

## Script
`countdown.sh`

### Purpose
Learned how a **while loop** works by repeatedly executing commands until a condition becomes false.

### Example

**Input**

```text
5
```

**Output**

```text
5
4
3
2
1
0
Done!
```

### Concepts Learned
- while loop
- Condition-based execution
- Updating variables inside a loop

---
# Task 3 – Command-Line Arguments

## Scripts
- `greet.sh`
- `args_demo.sh`

### Purpose

The goal of this task was to understand how a shell script can accept values while running instead of using hardcoded values. This makes scripts reusable because the same script can work with different inputs.

For example:

```bash
./greet.sh Rohit
```

Output

```text
Hello, Rohit!
```

Instead of editing the script every time, I can simply pass a different name while executing it.

---

### Concepts Learned

| Variable | Purpose |
|----------|----------|
| `$0` | Displays the script name |
| `$1` | First command-line argument |
| `$2` | Second command-line argument |
| `$#` | Total number of arguments |
| `$@` | Displays all arguments |

---

### Positive Example

```bash
./greet.sh Rohit
```

Output

```text
Hello, Rohit!
```

---

### Another Example

```bash
./args_demo.sh Linux Docker EC2
```

Output

```text
Script Name: ./args_demo.sh
Total Arguments: 3
All Arguments: Linux Docker EC2
```

---

### Mistakes I Made

Initially I executed:

```bash
./greet.sh Devops engg Rohit
```

I expected:

```text
Hello, Devops engg Rohit!
```

But the output was:

```text
Hello, Devops!
```

Reason:

Shell treats spaces as separators, so:

```
$1 = Devops
$2 = engg
$3 = Rohit
```

I learned that if I want the entire text as one argument, I should use quotes.

Correct command:

```bash
./greet.sh "Devops engg Rohit"
```

Output:

```text
Hello, Devops engg Rohit!
```

I also accidentally opened the wrong file (`grgs_demo.sh`) while editing and later corrected it.

---

### Real DevOps Use Case

A deployment script can receive the environment name as an argument.

Example:

```bash
./deploy.sh production
```

Instead of creating separate scripts for production, staging, and testing, one script can handle all environments using command-line arguments.

---

# Task 5 – Error Handling

## Script

`safe_script.sh`

### Purpose

The purpose of this task was to make shell scripts safer by handling errors gracefully instead of allowing the script to fail unexpectedly.

---

### Concepts Learned

- `set -e`
- `||` operator
- Error handling
- Safe scripting

---

### Positive Example

During the first execution:

```text
Script completed successfully
```

The script:

- Created the directory
- Entered the directory
- Created a file successfully

Everything worked as expected.

---

### Existing Directory Scenario

When I executed the script again, the directory already existed.

Output:

```text
Directory already exists
Script completed successfully
```

Instead of stopping abruptly, the script displayed a meaningful message and continued.

---

### Mistake I Learned

Without using:

```bash
||
```

the script would simply fail with:

```text
mkdir: cannot create directory
```

and the user would only see an error message.

Using:

```bash
mkdir /tmp/devops-test || echo "Directory already exists"
```

provides a much clearer explanation of what happened.

---

### Why Error Handling Matters

If a deployment or backup script fails unexpectedly in production, it becomes difficult to identify the issue.

Adding proper error handling makes scripts easier to debug and prevents unexpected failures.

---

### Real DevOps Use Case

While deploying an application, directories or configuration files may already exist.

Instead of failing, a well-written script detects the situation, informs the user, and safely continues wherever possible.

# Task 5 – Error Handling

## Script
`safe_script.sh`

### Purpose
Learned basic error handling techniques to make shell scripts safer and more reliable.

### Features
- Used `set -e` to stop execution if a command fails.
- Created a directory and file.
- Used the `||` operator to display custom error messages when needed.

### Example Output

```text
Script completed successfully
```

If the directory already exists:

```text
Directory already exists
Script completed successfully
```

### Concepts Learned
- `set -e`
- `||` operator
- Safe scripting practices

---

# 📸 Screenshots

- 01-for-loop-count.png
- 02-while-loop-countdown.png
- 03-command-line-arguments.png
- 04-install-packages-error-handling.png

---

# 🎯 Key Learnings

- Learned how to automate repetitive tasks using **for** and **while** loops.
- Understood how command-line arguments (`$0`, `$1`, `$#`, `$@`) make scripts reusable.
- Learned package installation automation, root privilege validation using `$EUID`, and basic error handling using `set -e` and `||`.

---

# Conclusion

Today's practice helped me understand how shell scripting can automate everyday DevOps tasks. I also learned how to write reusable and safer scripts using loops, command-line arguments, package validation, and error handling. These are practical concepts that will be useful in real-world DevOps environments.
