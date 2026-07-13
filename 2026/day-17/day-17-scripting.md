# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

---

## 👋 What is This Day About?

On Day 17, I leveled up my Shell Scripting skills by learning:

- **Loops** → Do the same task automatically without repeating commands
- **Command-Line Arguments** → Make scripts work with different inputs
- **Package Automation** → Install software automatically on any server
- **Error Handling** → Make scripts safe so they don't break silently

These are not just exercises — every DevOps engineer uses these concepts daily when writing automation scripts, deployment pipelines, and server setup tools.

---

## 📁 Scripts Written Today

| Script | What it Does |
|--------|-------------|
| `for_loop.sh` | Loops through a list of fruits |
| `count.sh` | Prints numbers 1 to 10 using a loop |
| `countdown.sh` | Counts down from any number to 0 |
| `greet.sh` | Greets a user by name passed as argument |
| `args_demo.sh` | Shows how bash reads command-line arguments |
| `install_packages.sh` | Auto-installs packages only if missing |
| `safe_script.sh` | Creates files safely with proper error handling |

---

# Task 1 – For Loop

## 🤔 What is a For Loop and WHY do we need it?

Imagine your manager says:

> *"Send a welcome email to 500 new users."*

Without a loop, you would write:
```bash
echo "Welcome, User1"
echo "Welcome, User2"
echo "Welcome, User3"
# ... 497 more lines 😭
```

With a loop, you write it **once** and it runs for everyone automatically:
```bash
for user in User1 User2 User3
do
    echo "Welcome, $user"
done
```

> **One rule → runs for every item. That's the power of a loop.**

---

## 🏭 Real-Time Scenario

**Situation:** You are a DevOps Engineer at a startup. Every Monday, you need to check the status of 10 services on your server — nginx, mysql, redis, etc.

**Without loop:** You run `systemctl status nginx`, then `systemctl status mysql`... 10 times manually every single Monday.

**With loop:**
```bash
for service in nginx mysql redis
do
    systemctl status $service
done
```

One script. Done in seconds. Every Monday. ✅

---

## 📝 `for_loop.sh`

```bash
#!/bin/bash
# Loops through 5 fruits and prints each one

for fruit in Apple Mango Banana Grapes Orange
do
    echo "Fruit: $fruit"
done
```

**How it works step by step:**
```
fruit = Apple   → prints "Fruit: Apple"
fruit = Mango   → prints "Fruit: Mango"
fruit = Banana  → prints "Fruit: Banana"
fruit = Grapes  → prints "Fruit: Grapes"
fruit = Orange  → prints "Fruit: Orange"
STOP — no more items
```

**✅ Actual Output from EC2 Terminal:**
```
$ touch for_loop.sh
$ nano for_loop.sh
$ chmod +x for_loop.sh
$ ./for_loop.sh
Fruit: Apple
Fruit: Mango
Fruit: Banana
Fruit: Grapes
Fruit: Orange
```

---

## 📝 `count.sh`

```bash
#!/bin/bash
# Prints numbers 1 to 10 using a for loop
# {1..10} is a shortcut — bash expands it to 1 2 3 4 5 6 7 8 9 10

for number in {1..10}
do
    echo "Number: $number"
done
```

> 💡 **Why `{1..10}`?** Instead of typing `1 2 3 4 5 6 7 8 9 10` manually, bash expands `{1..10}` automatically. You can even do `{1..1000}` with zero effort.

**✅ Actual Output from EC2 Terminal:**
```
$ touch count.sh
$ nano count.sh
$ chmod +x count.sh
$ ./count.sh
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
Number: 6
Number: 7
Number: 8
Number: 9
Number: 10
```

---

### 💡 Key Concept — For Loop

```
for VARIABLE in LIST
do
    # this runs once for each item in LIST
    # $VARIABLE holds the current item
done
```

Think of `VARIABLE` as an **empty box** 📦 — bash fills it with one item at a time, runs the block, then fills it with the next item.

---

# Task 2 – While Loop

## 🤔 What is a While Loop and WHY is it different from For Loop?

| For Loop | While Loop |
|----------|-----------|
| You know the list in advance | You don't know when to stop |
| `for fruit in Apple Mango` | `while num > 0` |
| Stops after last item | Stops when condition becomes false |

> **Simple rule:** Use `for` when you have a **fixed list**. Use `while` when you need to keep going **until something changes**.

---

## 🏭 Real-Time Scenario

**Situation:** You are monitoring a deployment. You want to keep checking if the app is running every 5 seconds — and stop only when it's finally up.

```bash
while ! curl -s http://myapp.com > /dev/null
do
    echo "App not ready yet... waiting"
    sleep 5
done
echo "App is UP! Deployment successful ✅"
```

This is exactly how **health check scripts** work in real DevOps pipelines. The script keeps checking — you don't know how long it will take, so you can't use a for loop.

---

## 📝 `countdown.sh`

```bash
#!/bin/bash
# Takes a number from the user and counts down to 0

echo "Enter a number:"
read num                        # read = wait for user to type, store in num

while [ $num -gt 0 ]           # -gt means "greater than"
do
    echo "$num"
    num=$((num - 1))           # reduce num by 1 each round — VERY IMPORTANT!
done

echo "Done!"
```

**How it works — if you type 4:**
```
num = 4 → is 4 > 0? YES → print 4 → num becomes 3
num = 3 → is 3 > 0? YES → print 3 → num becomes 2
num = 2 → is 2 > 0? YES → print 2 → num becomes 1
num = 1 → is 1 > 0? YES → print 1 → num becomes 0
num = 0 → is 0 > 0? NO  → STOP!
```

**✅ Actual Output from EC2 Terminal — Run 1 (entered 11):**
```
$ touch countdown.sh
$ nano countdown.sh
$ chmod +x countdown.sh
$ ./countdown.sh
Enter a number: 11
11
10
9
8
7
6
5
4
3
2
1
0
Done!
```

**✅ Actual Output from EC2 Terminal — Run 2 (entered 4):**
```
$ ./countdown.sh
Enter a number: 4
4
3
2
1
0
Done!
```

> ⚠️ **Important:** The line `num=$((num - 1))` reduces num by 1 every round. Without this line, num stays the same forever and the loop **never stops** — called an **infinite loop** 😱

---

### 💡 Comparison Operators in Bash

| Symbol | Meaning | Example |
|--------|---------|---------|
| `-gt` | greater than | `[ $num -gt 0 ]` |
| `-lt` | less than | `[ $num -lt 10 ]` |
| `-eq` | equal to | `[ $num -eq 5 ]` |
| `-ne` | not equal | `[ $num -ne 0 ]` |
| `-ge` | greater or equal | `[ $num -ge 1 ]` |

---

# Task 3 – Command-Line Arguments

## 🤔 What are Arguments and WHY do we need them?

Imagine you have a script that deploys your app. Without arguments:
```bash
# You have to edit the script every time — very bad practice!
environment="production"
```

With arguments:
```bash
./deploy.sh production    # pass it while running!
./deploy.sh staging
./deploy.sh testing
```

> **Arguments make your script reusable without editing it every time.**

---

## 🏭 Real-Time Scenario

**Situation:** You work at a company with 3 environments — production, staging, and testing. Every release, you deploy to all three. Instead of 3 separate scripts, you have ONE script that accepts the environment as an argument.

```bash
./deploy.sh production   # deploys to production
./deploy.sh staging      # deploys to staging
./deploy.sh testing      # deploys to testing
```

Same script. Different argument. Different result. That's the power of arguments.

---

## 🗺️ How Bash Reads Arguments

When you run `./greet.sh Rohit DevOps`:

```
$0  = ./greet.sh     ← the script name itself
$1  = Rohit          ← first word after script name
$2  = DevOps         ← second word
$#  = 2              ← total count of arguments
$@  = Rohit DevOps   ← all arguments together
```

> 💡 Think of `$1`, `$2` as **numbered slots** bash fills automatically when you run the script.

---

## 📝 `greet.sh`

```bash
#!/bin/bash
# Greets a user by name
# -z checks if $1 is empty (zero length)

if [ -z "$1" ]
then
    echo "Usage: ./greet.sh <name>"
    exit 1
fi

echo "Hello, $1!"
```

**✅ Actual Output from EC2 Terminal:**
```
$ touch greet.sh
$ nano greet.sh
$ chmod +x greet.sh

$ ./greet.sh Rohit
Hello, Rohit!

$ ./greet.sh Devops engg Rohit
Hello, Devops!

$ ./greet.sh
Usage: ./greet.sh <name>
```

---

### ❌ Mistake I Made — and What I Learned

I ran:
```bash
./greet.sh Devops engg Rohit
```

I expected `Hello, Devops engg Rohit!` but got:
```
Hello, Devops!
```

**Why?** Bash treats each space as a separator:
```
$1 = Devops
$2 = engg
$3 = Rohit
```
Script only uses `$1`, so it printed only `Devops`.

**Fix — use quotes:**
```bash
./greet.sh "Devops engg Rohit"
```
```
Hello, Devops engg Rohit!
```

> 💡 **Rule:** If your argument has spaces, always wrap it in **double quotes**.

---

## 📝 `args_demo.sh`

```bash
#!/bin/bash
# Demonstrates how bash reads what you pass to it

echo "Script Name  : $0"
echo "Total Args   : $#"
echo "All Args     : $@"
```

**✅ Actual Output from EC2 Terminal:**
```
$ touch args_demo.sh
$ nano args_demo.sh  (initially opened wrong file grgs_demo.sh by mistake)
$ chmod +x args_demo.sh

$ ./args_demo.sh Linux Docker EC2
Script Name: ./args_demo.sh
Total Arguments: 3
All Arguments: Linux Docker EC2
```

> 💡 **Honest mistake I made:** I accidentally typed `nano grgs_demo.sh` instead of `args_demo.sh` — created wrong file. I deleted it with `rm grgs_demo.sh` and then opened the correct file. Always double check your filename before typing!

---

### 💡 Argument Variables Quick Reference

| Variable | What it Contains | Example |
|----------|-----------------|---------|
| `$0` | Script name | `./greet.sh` |
| `$1` | First argument | `Rohit` |
| `$2` | Second argument | `DevOps` |
| `$#` | Total count | `2` |
| `$@` | All arguments | `Rohit DevOps` |

---

# Task 4 – Install Packages via Script

## 🤔 WHY automate package installation?

When you get a new server, you need to install tools — nginx, curl, wget, git, etc. Doing it manually every time is:
- Slow ⏱️
- Error-prone 🐛
- Not repeatable across multiple servers 😤

With a script, you run **one command** and everything installs automatically — same result every time on every server.

---

## 🏭 Real-Time Scenario

**Situation:** Your company just launched 50 new EC2 instances for a product launch. Each server needs nginx, curl, and wget. You can't SSH into 50 servers manually.

**Solution:** One script, runs on all 50 servers via automation:
```bash
bash install_packages.sh
```

All packages installed. All servers ready. In minutes. ✅

This is exactly how tools like **Ansible**, **Chef**, and **Puppet** work internally — they run scripts on multiple servers at once.

---

## 📝 `install_packages.sh`

```bash
#!/bin/bash

# ── Root Check ──────────────────────────────────────────────────
# $EUID = current user's ID. Root = 0. Non-root can't install packages.

if [ "$EUID" -ne 0 ]
then
    echo "Run this script as root."
    exit 1
fi

# ── Package List ─────────────────────────────────────────────────
packages=("nginx" "curl" "wget")

# ── Loop & Install ───────────────────────────────────────────────
for pkg in "${packages[@]}"
do
    if dpkg -s "$pkg" &> /dev/null
    then
        echo "$pkg is already installed."
    else
        echo "Installing $pkg..."
        apt install -y "$pkg"
        echo "$pkg installed."
    fi
done

echo "Done."
```

**✅ Actual Output from EC2 Terminal — Run as root:**
```
$ touch install_packages.sh
$ nano install_packages.sh
$ chmod +x install_packages.sh
$ ./install_packages.sh
nginx is already installed.
curl is already installed.
wget is already installed.
Done.

$ dpkg -s nginx
Package: nginx
Status: install ok installed
Priority: optional
Section: httpd
Installed-Size: 1322
Architecture: amd64
Version: 1.24.0-2ubuntu7.13
Description: small, powerful, scalable web/proxy server
```

**✅ Root check working — when run without sudo:**
```
$ ./install_packages.sh
Run this script as root.

$ sudo -i
root@ip-172-31-36-59:~#
```

> 💡 **What `dpkg -s nginx` confirms:** nginx is properly installed with version 1.24.0, status shows `install ok installed`. This proves our script correctly detected it was already installed and skipped re-installing it.

---

### 💡 Key Concepts

| Concept | What it Does |
|---------|-------------|
| `packages=("nginx" "curl")` | Array — stores multiple values |
| `"${packages[@]}"` | Expands all items in the array |
| `dpkg -s pkg` | Checks if package is installed |
| `&> /dev/null` | Hides command output completely |
| `apt install -y` | Installs without asking yes/no |
| `$EUID` | Current user's ID (0 = root) |

---

# Task 5 – Error Handling

## 🤔 WHY does Error Handling matter?

Imagine a deployment script with 10 steps. Step 3 fails — but the script keeps running steps 4, 5, 6... on a broken system. By step 10, everything is broken and you have **no idea where it went wrong**.

> **Without error handling → silent failures → debugging nightmare**
> **With error handling → fail fast → clear message → fix quickly**

---

## 🏭 Real-Time Scenario

**Situation:** You are deploying a Node.js app to production at 2 AM during a live sale. Your deployment script:

1. Creates a backup directory
2. Copies new files
3. Restarts the server

If step 1 fails (disk full) but script continues — step 2 copies to the wrong place, step 3 restarts with broken files. Your app crashes during peak traffic. 😱

**With error handling:**
```
❌ Failed to create backup directory — disk full
Script stopped. No changes made. Server safe.
```

You fix the disk, run again. Safe. ✅

---

## 📝 `safe_script.sh`

```bash
#!/bin/bash

set -e   # If ANY command fails → script stops immediately
         # Without this → broken steps keep running silently

echo "Creating directory..."
mkdir /tmp/devops-test || echo "Directory already exists"

cd /tmp/devops-test || { echo "Cannot enter directory. Stopping."; exit 1; }

touch testfile.txt || { echo "Cannot create file. Stopping."; exit 1; }

echo "Script completed successfully"
```

**✅ Actual Output from EC2 Terminal — First Run:**
```
$ touch safe_script.sh
$ nano safe_script.sh
$ chmod +x safe_script.sh
$ ./safe_script.sh
Script completed successfully
```

**✅ Second Run — Directory already exists:**
```
$ ./safe_script.sh
mkdir: cannot create directory '/tmp/devops-test': File exists
Directory already exists
Script completed successfully
```

**✅ Verified the file was created:**
```
$ ls /tmp/devops-test
testfile.txt
```

> 💡 **What happened on second run:** mkdir failed because `/tmp/devops-test` already existed. But instead of crashing, the `||` operator caught it and printed `Directory already exists`. Then script continued safely and completed successfully.

---

### 💡 Error Handling Toolkit

| Tool | What it Does | When to Use |
|------|-------------|-------------|
| `set -e` | Stop script if any command fails | Always — put at top of every script |
| `\|\|` | If left fails → run right | When failure is okay but needs a message |
| `&&` | If left succeeds → run right | Chain dependent commands |
| `{ }` | Group multiple commands | When failure should trigger multiple actions |
| `exit 1` | Exit with error code | Tell the system something went wrong |
| `exit 0` | Exit with success | Tell the system everything is fine |

---

### Before vs After Error Handling

**Without `||`:**
```
mkdir: cannot create directory '/tmp/devops-test': File exists
```
Script crashes. No context. User confused. 😤

**With `||`:**
```
Directory already exists
Script completed successfully
```
Script handles it. User informed. Continues safely. ✅

---

# 🎯 3 Key Learnings from Day 17

### 1. Loops eliminate repetition

Before loops, you copy-paste the same command 10, 50, 100 times. A single loop handles any number of items with the same code. This is the foundation of all automation scripts in DevOps.

### 2. Arguments make scripts reusable

A hardcoded script works once. A script with arguments works forever. The same deploy script can handle production, staging, and testing — just pass a different argument. This is how real DevOps tools work.

### 3. Error handling is not optional in production

Scripts without error handling are dangerous in production. A silent failure at step 2 can corrupt everything by step 10. `set -e` and `||` make failures loud and early — which is always better than silent and late.

---

# 📊 Summary Table

| Task | Script | Key Concept | Real DevOps Use |
|------|--------|-------------|----------------|
| For Loop | `for_loop.sh`, `count.sh` | Iterate over list | Check multiple services at once |
| While Loop | `countdown.sh` | Loop until condition false | Health check until app is ready |
| Arguments | `greet.sh`, `args_demo.sh` | `$1 $2 $# $@` | Deploy to different environments |
| Package Install | `install_packages.sh` | Auto-install + root check | Bootstrap new servers automatically |
| Error Handling | `safe_script.sh` | `set -e` and `\|\|` | Safe production deployments |

---

# Conclusion

Day 17 showed me that shell scripting is not just about running commands — it's about writing scripts that are **smart**, **reusable**, and **safe**. Loops remove repetition. Arguments add flexibility. Error handling adds reliability. Together, these make the difference between a beginner script and a production-ready automation tool.

I also learned from my own mistakes — like accidentally opening the wrong file (`grgs_demo.sh` instead of `args_demo.sh`), forgetting `+x` in chmod, and understanding why `num=$((num-1))` is critical in a while loop. These small errors taught me more than getting it right the first time.

---
