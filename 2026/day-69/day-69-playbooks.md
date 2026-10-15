# Day 69 – Ansible Playbooks, Modules, Handlers and Multi-Play

> **Goal of the day:** Learn how to write Ansible playbooks, use the most common modules, handle errors, use handlers, debug runs, and target different server groups with different plays.
>
> **Setup used:** 3 servers managed from one control node:
> `web-server`, `app-server`, `db-server`.
>
> **Note:** The playbook code below is the standard version of each task. If your own file has slightly different names, paths or IPs, keep your version and just follow the explanation.

---

## 0. Basic Concepts (read this first)

| Term | Simple meaning |
|---|---|
| **Control node** | The machine where you run `ansible` commands. |
| **Managed node** | The servers Ansible configures (web, app, db). |
| **Inventory** | A file that lists your servers and groups them. |
| **Playbook** | A YAML file that says *what to do* on *which servers*. |
| **Play** | One block in a playbook: "on these hosts, run these tasks". |
| **Task** | One single action (install a package, copy a file, etc). |
| **Module** | The tool a task uses (`apt`, `service`, `copy`...). |
| **Handler** | A special task that runs only when another task reports a change (for example, restart Nginx). |
| **Idempotent** | Running it again gives the same result and changes nothing new. |

### Inventory used in this project

```ini
[web]
web-server

[app]
app-server

[db]
db-server
```

Each server has its own `ansible_host`, `ansible_user` and key in the real inventory file.

### Output colors you will see
- **green `ok`**: nothing needed to change
- **yellow `changed`**: Ansible changed something
- **red `failed`**: task failed
- **blue `skipped`**: task was skipped

### Folder structure for this day

```
2026/day-69/
├── day-69.md
└── Screenshots/
    ├── task1-*.png
    ├── ...
    └── task6-*.png
```

---

## Task 1: Your First Playbook (Install and Start Nginx)

**What:** Check connection to all servers, write a first playbook, run it twice.

**Why:** A playbook is how you turn manual server work into repeatable code. Running it twice proves *idempotency*.

### Step 1: Check inventory and connection

```bash
ansible all -m ping
```

- `ansible` = ad-hoc command tool
- `all` = every host in the inventory
- `-m ping` = use the `ping` module (this is **not** network ping, it checks that Ansible can log in and run Python on the server)

Expected: every server replies `"ping": "pong"` in green.

![Inventory ping](Screenshots/task1-01-inventory-ping.png)

### Step 2: Write the playbook `install-nginx.yml`

```yaml
---
- name: Install and start Nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Make sure Nginx is running and enabled
      service:
        name: nginx
        state: started
        enabled: yes
```

**Line by line:**
- `---` starts a YAML file.
- `name:` a human-readable label (shows in output).
- `hosts: web` run only on the `web` group.
- `become: true` use `sudo` (root rights), needed to install packages.
- `apt` module installs packages on Ubuntu/Debian. `state: present` means "make sure it is installed".
- `service` module controls services. `started` = running now, `enabled` = starts on boot.

### Step 3: Syntax check (no changes are made)

```bash
ansible-playbook install-nginx.yml --syntax-check
```

Expected: `playbook: install-nginx.yml`. This only checks YAML/Ansible grammar.

![Syntax check](Screenshots/task1-01-playbook-syntax-check.png)

### Step 4: First run

```bash
ansible-playbook install-nginx.yml
```

Expected: tasks show **changed** (Nginx got installed and started).

![First run](Screenshots/task1-02-first-run.png)

### Step 5: Second run (idempotency)

```bash
ansible-playbook install-nginx.yml
```

Expected: everything is **ok**, `changed=0`. Nothing was needed because the server was already in the desired state.

![Second run, idempotent](Screenshots/task1-03-second-run-idempotent.png)

### Step 6: Verify with curl

```bash
ansible web -m shell -a "curl -s localhost | head -5"
```

Expected: the Nginx welcome page HTML.

![Curl verify](Screenshots/task1-04-curl-verify.png)

**Key learning:** Playbooks describe the *desired state*, not step-by-step commands.

---

## Task 2: Error Handling (fail and ignore_errors)

**What:** Make a task fail on purpose, then learn to ignore the failure.

**Why:** In real life, some commands fail and you must decide: stop everything, or continue.

### Step 1: Playbook where a task fails (`fail-demo.yml`)

```yaml
---
- name: Fail demo
  hosts: web
  tasks:
    - name: This task will fail
      command: /bin/false

    - name: This task will NOT run
      debug:
        msg: "You will not see this"
```

- `/bin/false` always exits with an error.
- By default Ansible **stops** the play for that host when a task fails.

```bash
ansible-playbook fail-demo.yml
```

![Fail demo playbook](Screenshots/task2-01-fail-demo-playbook.png)
![Fail demo run](Screenshots/task2-02-fail-demo-run.png)

### Step 2: Use `ignore_errors` (`ignore-errors.yml`)

```yaml
---
- name: Ignore errors demo
  hosts: web
  tasks:
    - name: This task fails but we ignore it
      command: /bin/false
      ignore_errors: yes

    - name: This task still runs
      debug:
        msg: "Playbook continued after the failure"
```

```bash
ansible-playbook ignore-errors.yml
```

Expected: you see `...ignoring` in the output and the second task runs.

![Ignore errors playbook](Screenshots/task2-03-ignore-errors-playbook.png)
![Ignore errors result](Screenshots/task2-04-ignore-errors-result.png)

**Key learning:** Use `ignore_errors: yes` only when a failure is acceptable. Do not hide real problems.

---

## Task 3: Core Modules (apt, service, copy, file, command, shell, lineinfile)

**What:** Practice the 7 most used modules, then run them all on all servers.

**Why:** Almost all daily Ansible work is done with these modules.

### 3.1 `apt` – install or remove packages

```yaml
- name: Install packages
  apt:
    name:
      - git
      - curl
    state: present
    update_cache: yes
```

`state: present` = install, `absent` = remove, `latest` = upgrade.

![apt module](Screenshots/task3-01-apt-module.png)

### 3.2 `service` – manage services

```yaml
- name: Start and enable Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

`started`, `stopped`, `restarted`, `reloaded` are the common states.

![service playbook](Screenshots/task3-02-service-playbook.png)
![service result](Screenshots/task3-03-service-result.png)

### 3.3 `copy` – copy a file or write content

```yaml
- name: Create a welcome file
  copy:
    content: "Deployed by Ansible\n"
    dest: /tmp/welcome.txt
    mode: '0644'
```

`content` writes text directly. Use `src:` to copy a file from the control node.

![copy playbook](Screenshots/task3-04-copy-playbook.png)
![copy result](Screenshots/task3-05-copy-result.png)

### 3.4 `file` – create folders, set permissions

```yaml
- name: Create app directory
  file:
    path: /opt/app
    state: directory
    mode: '0755'
```

`state` can be `directory`, `file`, `touch`, `absent` (delete).

![file module](Screenshots/task3-06-file-module.png)

### 3.5 `command` – run a command (no shell features)

```yaml
- name: Check disk space
  command: df -h /
  register: disk_output

- name: Show output
  debug:
    var: disk_output.stdout_lines
```

- `register` saves the result in a variable.
- `command` does **not** support pipes (`|`), `>` or `||`.

![command playbook](Screenshots/task3-07-command-playbook.png)
![command result](Screenshots/task3-08-command-result.png)

### 3.6 `shell` – run a command with shell features

```yaml
- name: Count files in /etc
  shell: ls /etc | wc -l
  register: etc_count
```

`shell` allows pipes, redirects and `||`. It always shows **changed** because Ansible cannot know if a shell command changed something.

![shell playbook](Screenshots/task3-09-shell-playbook.png)
![shell result](Screenshots/task3-10-shell-result.png)

> **command vs shell:** use `command` by default (safer). Use `shell` only when you need pipes or redirects.

### 3.7 `lineinfile` – edit one line in a file

```yaml
- name: Add a line to a file
  lineinfile:
    path: /tmp/welcome.txt
    line: "Managed by Ansible"
    create: yes
```

It adds the line only if it is missing, so it is idempotent.

![lineinfile playbook](Screenshots/task3-11-lineinfile-playbook.png)
![lineinfile result](Screenshots/task3-12-lineinfile-result.png)

### 3.8 Run everything on all servers

All the above tasks are combined in one playbook and run with:

```bash
ansible-playbook all-servers.yml
```

The run is long, so it is split in 3 screenshots plus the recap.

![All servers part 1](Screenshots/task3-17-all-servers-run-part1.png)
![All servers part 2](Screenshots/task3-18-all-servers-run-part2.png)
![All servers part 3](Screenshots/task3-19-all-servers-run-part3.png)
![All servers recap](Screenshots/task3-20-all-servers-recap.png)

**Key learning:** Each module has a `state` or similar option that describes the *end result* you want.

---

## Task 4: Handlers (Restart Only When Needed)

**What:** Push an Nginx config and restart Nginx only if the config really changed.

**Why:** Restarting a service on every run is wasteful and can cause downtime. Handlers solve this.

### Playbook (`nginx-config.yml`)

```yaml
---
- name: Configure Nginx with a handler
  hosts: web
  become: true

  tasks:
    - name: Deploy custom index page
      copy:
        content: "<h1>Hello from Ansible</h1>\n"
        dest: /var/www/html/index.html
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**How it works:**
- `notify: Restart Nginx` is a signal. It fires **only if the task is `changed`**.
- The handler runs **once, at the end** of the play, even if notified many times.
- If the file is already correct, the task is `ok`, no notify, no restart.

![Nginx config playbook](Screenshots/task4-02-nginx-config-playbook.png)

### First run

```bash
ansible-playbook nginx-config.yml
```

Expected: task is `changed` and you see `RUNNING HANDLER [Restart Nginx]`.

![First run with handler](Screenshots/task4-03-first-run-handler.png)

### Verify

```bash
ansible web -m shell -a "curl -s localhost"
```

Expected: `<h1>Hello from Ansible</h1>`.

![Curl verify](Screenshots/task4-05-curl-verify.png)

> If you run the playbook again, the handler will **not** run, because nothing changed.

---

## Task 5: Dry Run, Diff, Verbose Mode, Limit and List

**What:** Learn the options that help you test and debug playbooks safely.

**Why:** Before touching production you should preview changes and know how to see more detail when something breaks.

### 5.1 Check mode (dry run)

```bash
ansible-playbook nginx-config.yml --check
```

Shows what **would** change, but changes nothing.

![Check mode](Screenshots/task5-01-check-mode.png)

### 5.2 Check + Diff

```bash
ansible-playbook nginx-config.yml --check --diff
```

`--diff` shows the exact lines that would be added (`+`) or removed (`-`).

![Check and diff](Screenshots/task5-02-check-diff.png)

### 5.3 Verbose levels

```bash
ansible-playbook install-nginx.yml -v
ansible-playbook install-nginx.yml -vv
ansible-playbook install-nginx.yml -vvv
```

| Flag | What you get |
|---|---|
| `-v` | Task results (return values) |
| `-vv` | Plus which playbook file and task path are used |
| `-vvv` | Plus SSH connection details (very useful for login problems) |

![-v part 1](Screenshots/task5-03-verbose-v-part1.png)
![-v part 2](Screenshots/task5-04-verbose-v-part2.png)
![-vv part 1](Screenshots/task5-05-verbose-vv-part1.png)
![-vv part 2](Screenshots/task5-06-verbose-vv-part2.png)
![-vv part 3](Screenshots/task5-07-verbose-vv-part3.png)
![-vvv part 1](Screenshots/task5-08-verbose-vvv-part1.png)
![-vvv part 2](Screenshots/task5-09-verbose-vvv-part2.png)

### 5.4 Limit to specific hosts

```bash
ansible-playbook all-servers.yml --limit web-server
```

Runs the playbook only on `web-server`, even though the playbook says `hosts: all`.

![Limit](Screenshots/task5-10-limit.png)

### 5.5 List hosts and tasks (nothing is executed)

```bash
ansible-playbook all-servers.yml --list-hosts
ansible-playbook all-servers.yml --list-tasks
```

- `--list-hosts` shows which servers would be targeted.
- `--list-tasks` shows all tasks in order.

![List hosts and tasks](Screenshots/task5-11-list-hosts-tasks.png)

**Key learning:** Always do `--syntax-check`, then `--check --diff`, then the real run.

---

## Task 6: Multi-Play Playbook (Different Work for Different Groups)

**What:** One playbook with 3 plays. Each play targets a different group and installs different software.

**Why:** Real infrastructure has different roles. One playbook can configure all of them correctly.

### Playbook (`multi-play.yml`)

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes

- name: Configure app servers
  hosts: app
  become: true
  tasks:
    - name: Install build tools
      apt:
        name:
          - gcc
          - make
        state: present
        update_cache: yes

    - name: Create app directory
      file:
        path: /opt/app
        state: directory
        mode: '0755'

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    - name: Install MySQL client
      apt:
        name: mysql-client
        state: present
        update_cache: yes

    - name: Create data directory
      file:
        path: /var/lib/appdata
        state: directory
        mode: '0755'
```

![Multi-play playbook](Screenshots/task6-01-multi-play-playbook.png)

### Step 1: Syntax check

```bash
ansible-playbook multi-play.yml --syntax-check
```

### Step 2: Remove Nginx from app and db servers

Nginx was installed on all 3 servers earlier (Task 3). To prove that each play only works on its own group, remove it from `app` and `db`:

```bash
ansible app,db -b -m apt -a "name=nginx state=absent"
```

- `app,db` = target both groups
- `-b` = become (sudo)
- `-m apt` = use the apt module
- `-a "..."` = module arguments

![Syntax check and Nginx removal](Screenshots/task6-02-syntax-check-nginx-remove-part1.png)

### Step 3: Confirm the starting state

```bash
ansible all -m shell -a "which nginx || echo not-installed"
```

Result: Nginx only on `web-server`. The other two show `not-installed`.
(`shell` shows CHANGED, this is normal as learned in Task 3.)

![Nginx removed, verify](Screenshots/task6-03-nginx-removed-verify.png)

### Step 4: Run the multi-play playbook

```bash
ansible-playbook multi-play.yml
```

You will see 3 separate plays:
- `PLAY [Configure web servers]` → only `web-server`
- `PLAY [Configure app servers]` → only `app-server`
- `PLAY [Configure database servers]` → only `db-server`

![Multi-play run](Screenshots/task6-04-multi-play-run-part1.png)

### Step 5: Read the PLAY RECAP

```
app-server : ok=3 changed=2 failed=0
db-server  : ok=3 changed=2 failed=0
web-server : ok=3 changed=0 failed=0
```

- `web-server`: `changed=0` because Nginx was already installed and running.
- `app-server`: `changed=2` because gcc/make were installed and `/opt/app` was created.
- `db-server`: `changed=2` because mysql-client was installed and `/var/lib/appdata` was created.
- `failed=0` on all servers.
- `ok=3` on each server = **Gathering Facts + its own 2 tasks**. Tasks of other groups never ran there. That is the whole point of multi-play.

![Multi-play recap](Screenshots/task6-05-multi-play-recap.png)

### Step 6: Verify

```bash
ansible all -m shell -a "which nginx || echo not-installed"
ansible all -m shell -a "which mysql || echo not-installed"
```

| Check | web-server | app-server | db-server |
|---|---|---|---|
| nginx | `/usr/sbin/nginx` | not-installed | not-installed |
| mysql | not-installed | not-installed | `/usr/bin/mysql` |

This proves every play changed only its own group.

![Verify nginx and mysql](Screenshots/task6-06-verify-nginx-mysql.png)

---

## Command Cheat Sheet

| Command | Purpose |
|---|---|
| `ansible all -m ping` | Test connection to all hosts |
| `ansible <group> -m <module> -a "<args>"` | Run one module ad-hoc |
| `ansible-playbook file.yml` | Run a playbook |
| `--syntax-check` | Validate the syntax only |
| `--check` | Dry run, no changes |
| `--diff` | Show line-level changes |
| `-v / -vv / -vvv` | More detailed output |
| `--limit <host>` | Run on selected hosts only |
| `--list-hosts` | Show target hosts |
| `--list-tasks` | Show tasks in order |

## Module Cheat Sheet

| Module | Use | Common option |
|---|---|---|
| `ping` | Test connectivity | none |
| `apt` | Install/remove packages | `state: present/absent` |
| `service` | Start/stop/enable services | `state`, `enabled` |
| `copy` | Copy file or write text | `src/content`, `dest` |
| `file` | Folders and permissions | `state: directory`, `mode` |
| `command` | Run a simple command | `register` |
| `shell` | Command with pipes/redirects | `register` |
| `lineinfile` | Add/replace one line | `line`, `path` |

---

## Key Takeaways

1. A playbook describes the **desired state**, and Ansible makes the server match it.
2. Running it again changes nothing (**idempotent**).
3. Tasks stop on failure unless you use `ignore_errors`.
4. **Handlers** run only when something changed, which avoids useless restarts.
5. Use `--syntax-check`, `--check --diff` and `-v` before and during real runs.
6. **Multi-play** playbooks let one file configure many server roles.
7. `command` is safer than `shell`. Use `shell` only when you need shell features.

## How to Practice This Day

1. Create the 3-server inventory with `web`, `app`, `db` groups.
2. Run `ansible all -m ping` until all servers are green.
3. Do Task 1 to Task 6 in order, using the commands above.
4. Take a screenshot after each step and save it in `Screenshots/` with the same file names used in this document.
5. Commit and push to GitHub.
