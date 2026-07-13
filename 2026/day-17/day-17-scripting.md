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
Learned how to pass values while running a shell script, making scripts more flexible and reusable.

### Concepts Learned
- `$0` → Script name
- `$1` → First argument
- `$#` → Total number of arguments
- `$@` → All arguments

### Example

```bash
./greet.sh Rohit
```

**Output**

```text
Hello, Rohit!
```

---

# Task 4 – Install Packages via Script

## Script
`install_packages.sh`

### Purpose
Created a script to automate package installation by checking whether packages are already installed before installing them.

### Features
- Checks if a package is installed.
- Installs missing packages.
- Skips packages that are already installed.
- Verifies root privileges using `$EUID`.

### Packages Used
- nginx
- curl
- wget

### Concepts Learned
- `$EUID`
- `dpkg -s`
- Package automation
- Root user validation

---

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
