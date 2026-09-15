# Day 51 – Kubernetes Manifests and Your First Pods

This guide is written so that even a person with **zero knowledge of Kubernetes** can read it, follow it, and do the exact same tasks. Every command is explained word by word, in simple English. No prior experience needed.

---

## Before We Start: What is a Pod?

A **Pod** is the smallest unit you can create in Kubernetes. Think of a Pod as a small box that holds one (or sometimes more) containers. A container is basically a running application (like a web server, or a database) packaged with everything it needs to run.

## What is a Manifest?

A **manifest** is just a text file (written in a format called YAML) where you describe what you want Kubernetes to create. You don't tell Kubernetes "run these exact steps" — you just describe "this is what I want to exist," and Kubernetes figures out how to make it happen.

## The 4 Required Parts of Every Manifest

Every manifest file needs these 4 top-level sections:

```yaml
apiVersion: v1          # Which "version" of the Kubernetes API to talk to
kind: Pod               # What type of thing you are creating
metadata:               # The "identity card" of the thing you are creating
  name: my-pod
  labels:
    app: my-app
spec:                   # What you actually want it to do
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

Word by word:
- **apiVersion** — Kubernetes has different "groups" of features. `v1` is the oldest and most basic group, used for simple things like Pods. Later, more advanced things like Deployments use `apps/v1`.
- **kind** — This tells Kubernetes what kind of object you are making. Today it's always `Pod`.
- **metadata** — This is information ABOUT the object, not what it does. `name` is required and must be unique. `labels` are tags (key-value pairs) you can attach for organizing things later.
- **spec** — This is the actual instructions: what containers to run, which image (software) to use, which network port to open.

---

# TASK 1: Create Your First Pod (Nginx)

**Goal:** Create a Pod running Nginx (a web server), and prove it works by visiting it from inside the container.

## Step 1: Create the manifest file

Open a text editor in the terminal:
```bash
nano nginx-pod.yaml
```
Word by word: `nano` is a simple text editor that runs inside the terminal. `nginx-pod.yaml` is the name of the new file you are creating.

## Step 2: Type this content into the file

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Explanation of every line:
- `apiVersion: v1` — Use the basic/core API group (Pods always use this).
- `kind: Pod` — We are creating a Pod.
- `metadata:` — Start of the identity section.
- `name: nginx-pod` — The Pod's unique name is "nginx-pod". You will use this name later in every command that refers to this Pod.
- `labels:` — Start of the tags section.
- `app: nginx` — A tag saying "this belongs to the nginx app". This is just for organizing/filtering later, it doesn't change how the Pod runs.
- `spec:` — Start of the "what to actually do" section.
- `containers:` — A list of containers to run inside this Pod (here, just one).
- `- name: nginx` — The dash (`-`) means "this is one item in the list". This container's name is "nginx" (different from the Pod's name above).
- `image: nginx:latest` — Use the Docker image called `nginx`, with the tag `latest` (meaning the newest available version).
- `ports:` — A list of ports this container uses.
- `- containerPort: 80` — This container listens on port 80. **Important: this line is just documentation/information — it does NOT open the port to the outside world by itself.**

Save the file: press `Ctrl+O`, then `Enter`, then `Ctrl+X` to exit.

## Step 3: Apply the manifest (create the Pod)

```bash
kubectl apply -f nginx-pod.yaml
```
Word by word:
- `kubectl` — The command-line tool used to talk to a Kubernetes cluster.
- `apply` — Take whatever is in the file and make the cluster match it (create it if it doesn't exist, update it if it does).
- `-f` — Stands for "file". Tells kubectl "the next word is a filename to read".
- `nginx-pod.yaml` — The file to read.

Expected output:
```
pod/nginx-pod created
```

## Step 4: Check if the Pod is running

```bash
kubectl get pods
```
Word by word: `get` means "show me a list of". `pods` is the type of resource to list.

Expected output (once it starts up):
```
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          33s
```
- `READY 1/1` — 1 out of 1 containers in this Pod are ready.
- `STATUS Running` — The container has started successfully.
- `RESTARTS 0` — It has never crashed.
- `AGE` — How long ago it was created.

For more detail (including which server/Node it's running on and its internal IP address):
```bash
kubectl get pods -o wide
```
`-o wide` means "show output in the wide/detailed format".

## Step 5: Look at detailed information about the Pod

```bash
kubectl describe pod nginx-pod
```
`describe` shows a big, detailed report about one specific resource. `pod nginx-pod` means "the Pod named nginx-pod".

This shows things like:
- Which Node (server) it's running on
- Its internal IP address
- A list of **Events** at the bottom — this is the step-by-step history of what happened (Scheduled → Pulling image → Pulled → Created → Started). This is the first place to look when something goes wrong.

## Step 6: Read the container's logs

```bash
kubectl logs nginx-pod
```
`logs` shows you the text output that the container has printed since it started (like a diary of what it's been doing).

## Step 7: Go inside the container and test it yourself

```bash
kubectl exec -it nginx-pod -- /bin/bash
```
Word by word:
- `exec` — Run a new command inside an already-running container.
- `-it` — Two options combined: `-i` (interactive, meaning your keyboard input goes into the container) and `-t` (gives you a proper terminal screen).
- `nginx-pod` — Which Pod to go into.
- `--` — Everything after this is the actual command to run inside the container.
- `/bin/bash` — Start a bash shell (a command prompt) inside the container.

After this, your terminal prompt changes — you are now "inside" the container.

Now, still inside the container, test the web server:
```bash
curl localhost:80
```
`curl` sends a web request. `localhost:80` means "send it to this same machine, on port 80" (since you are now inside the same container where Nginx is running).

**Expected result:** You should see HTML starting with `<!DOCTYPE html>` and the text "Welcome to nginx!" This proves Nginx is working correctly.

Leave the container:
```bash
exit
```

### Task 1 Screenshots

**Applying the pod, checking status:**
![Task 1 - Nginx Pod Running](2026/day-51/Screenshots/day-51-task1-nginx-pod-running.png)

**Detailed describe output (part 1):**
![Task 1 - Describe Pod Part 1](2026/day-51/Screenshots/day-51-task1-describe-pod-part1.png)

**Detailed describe output (part 2):**
![Task 1 - Describe Pod Part 2](2026/day-51/Screenshots/day-51-task1-describe-pod-part2.png)

**Logs, going inside the container, and curl showing the Nginx welcome page:**
![Task 1 - Logs and Curl](2026/day-51/Screenshots/day-51-task1-logs-and-curl.png)

---

# TASK 2: Create a Custom Pod (BusyBox)

**Goal:** Create a second Pod using a different kind of image (BusyBox), and understand why it needs a special `command` field to stay running.

BusyBox is a tiny Linux toolbox image — it does not run a server on its own like Nginx does. If you don't tell it to do something that takes time, it will finish instantly and the container will stop.

## Step 1: Create a new file

```bash
nano busybox-pod.yaml
```

## Step 2: Type this content

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

Explanation of the new/different parts:
- `labels:` now has **two** tags: `app: busybox` and `environment: dev`. A resource can have as many labels as you want.
- `command: [...]` — This tells the container exactly what program to run when it starts, instead of using whatever the image's default is. It is written as a list (using square brackets `[ ]`), where each item is one word/argument:
  - `"sh"` — Run the "sh" shell program.
  - `"-c"` — Tells the shell "the next item is a command string to execute".
  - `"echo Hello from BusyBox && sleep 3600"` — This is actually two commands joined together:
    - `echo Hello from BusyBox` — Print the text "Hello from BusyBox" (this will show up in the logs).
    - `&&` — Means "if the first command succeeded, then run the next one".
    - `sleep 3600` — Wait for 3600 seconds (1 hour), doing nothing. This is what keeps the container alive/running.

**Why this matters:** If you didn't add `sleep 3600`, the container would print "Hello" and immediately exit. Kubernetes would then try to restart it automatically (because its default restart policy is "Always"). If it keeps exiting immediately over and over, Kubernetes will eventually show the status `CrashLoopBackOff` — meaning "this keeps crashing, so I'm going to wait longer between retries."

Save the file (`Ctrl+O`, `Enter`, `Ctrl+X`).

## Step 3: Apply it

```bash
kubectl apply -f busybox-pod.yaml
```
Expected output: `pod/busybox-pod created`

## Step 4: Check the status

```bash
kubectl get pods
```
You should now see both `nginx-pod` and `busybox-pod` in the list, both with status `Running` and `RESTARTS: 0`.

## Step 5: Check the logs to confirm it worked

```bash
kubectl logs busybox-pod
```
Expected output:
```
Hello from BusyBox
```
This confirms the `command` field worked correctly.

### Task 2 Screenshot

**Full manifest, apply, get pods, and logs showing "Hello from BusyBox":**
![Task 2 - BusyBox Pod](2026/day-51/Screenshots/day-51-task2-busybox-pod.png)

---

# TASK 3: Imperative vs Declarative

**Goal:** Learn the two different ways of creating things in Kubernetes, and a trick to quickly generate a starter manifest.

There are two styles of working with Kubernetes:

- **Declarative** (what you did in Task 1 and 2): You write a YAML file describing what you want, and run `kubectl apply -f file.yaml`. This file can be saved, reused, and tracked in version control (like Git). This is the recommended way for real projects.
- **Imperative**: You type a direct command and Kubernetes creates the thing immediately — no file involved. Quick for testing, but nothing gets saved anywhere for later.

## Step 1: Create a Pod the imperative way (no YAML file at all)

```bash
kubectl run redis-pod --image=redis:latest
```
Word by word:
- `run` — Create and run a Pod right now (imperative command).
- `redis-pod` — The name to give this new Pod.
- `--image=redis:latest` — Which container image to use.

Expected output: `pod/redis-pod created`

## Step 2: Check it exists

```bash
kubectl get pods
```
You should now see `nginx-pod`, `busybox-pod`, AND `redis-pod` all running.

## Step 3: Extract the YAML that Kubernetes generated automatically

Even though you didn't write any YAML for `redis-pod`, Kubernetes actually created a full YAML definition behind the scenes. You can see it:

```bash
kubectl get pod redis-pod -o yaml
```
Word by word: `-o yaml` means "show me the output in YAML format" (instead of the short table we normally see).

This output will be MUCH longer than anything you wrote by hand. It includes everything you wrote, plus a lot of extra information that Kubernetes fills in automatically:

- Under `metadata`: `creationTimestamp` (exact time it was created), `namespace: default` (you didn't specify one, so it used the default), `resourceVersion` and `uid` (internal tracking IDs).
- Under `spec`: `dnsPolicy`, `restartPolicy: Always`, `nodeName` (which server it's running on), `schedulerName`, `terminationGracePeriodSeconds` (how long to wait before force-stopping it), and an automatically-attached "volume" giving the Pod permission to talk to the Kubernetes API.
- A whole `status:` section — this describes what is ACTUALLY happening right now (is it running? what's its IP address? is it ready?). You never write this part yourself — Kubernetes fills it in and keeps it updated live.

**Key idea:** The YAML you write by hand is only the "wish list" (what you want). What you see with `-o yaml` is the full real picture — your wish list PLUS all the defaults Kubernetes filled in PLUS live status information.

## Step 4: Use "dry-run" to preview a YAML file without creating anything

```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml
```
Word by word:
- `--dry-run=client` — "Don't actually create anything. Just show me what WOULD happen, calculated only on my own computer (the 'client'), without contacting the cluster."
- `-o yaml` — Show the result as YAML text.

This is very useful: instead of writing a whole manifest from scratch and possibly getting the structure wrong, you can generate a correct starting template instantly, then edit it to add whatever extra settings you need.

## Step 5: Save that generated YAML into an actual file

```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml > test-pod-dryrun.yaml
```
The `>` symbol means "don't print this on screen — save it into a file instead". Nothing will appear on the screen when you run this; that is normal.

Check the saved file:
```bash
cat test-pod-dryrun.yaml
```
`cat` just prints the full contents of a file to the screen.

### Task 3 Screenshots

**Creating redis-pod imperatively and viewing full generated YAML (part 1):**
![Task 3 - Redis Run and YAML Part 1](2026/day-51/Screenshots/day-51-task3-redis-run-and-yaml-part1.png)

**Full generated YAML continued (part 2):**
![Task 3 - Redis YAML Part 2](2026/day-51/Screenshots/day-51-task3-redis-yaml-part2.png)

**Full generated YAML — status section (part 3):**
![Task 3 - Redis YAML Part 3 Status](2026/day-51/Screenshots/day-51-task3-redis-yaml-part3-status.png)

**Full generated YAML — status section continued (part 4):**
![Task 3 - Redis YAML Part 4 Status](2026/day-51/Screenshots/day-51-task3-redis-yaml-part4-status.png)

**Dry-run output on screen:**
![Task 3 - Dry-run Output](2026/day-51/Screenshots/day-51-task3-dryrun-output.png)

**Dry-run output saved into a file:**
![Task 3 - Dry-run Saved to File](2026/day-51/Screenshots/day-51-task3-dryrun-saved-to-file.png)

---

# TASK 4: Validate Before Applying

**Goal:** Learn how to check a YAML file for mistakes BEFORE actually creating anything, and see what error messages look like.

## Step 1: Client-side dry-run on your existing file

```bash
kubectl apply -f nginx-pod.yaml --dry-run=client
```
This just checks that the file's structure/syntax is written correctly. It does NOT contact the Kubernetes API server for deep checking.

Expected output (since the Pod already exists and nothing changed):
```
pod/nginx-pod unchanged (dry run)
```

## Step 2: Server-side dry-run on the same file

```bash
kubectl apply -f nginx-pod.yaml --dry-run=server
```
This time, the request IS actually sent to the Kubernetes API server for full checking, but nothing is actually saved/created at the end.

Expected output:
```
pod/nginx-pod unchanged (server dry run)
```

## Step 3: Intentionally break the file — remove the `image` field

First, make a copy so your working file stays safe:
```bash
cp nginx-pod.yaml broken-pod.yaml
```
`cp` means "copy". This copies `nginx-pod.yaml` into a new file called `broken-pod.yaml`.

Edit the copy:
```bash
nano broken-pod.yaml
```
Delete the line `image: nginx:latest` completely, so the file now looks like:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    ports:
    - containerPort: 80
```
Save it (`Ctrl+O`, `Enter`, `Ctrl+X`).

## Step 4: Try client-side dry-run on the broken file

```bash
kubectl apply -f broken-pod.yaml --dry-run=client
```
**Surprising result:** No error appears! It says `pod/nginx-pod configured (dry run)` as if everything is fine.

**Why?** Because `--dry-run=client` only checks basic YAML formatting (indentation, brackets, etc.) — it does not know Kubernetes' actual rule that `image` is required. It never asks the real API server.

## Step 5: Try server-side dry-run on the broken file

```bash
kubectl apply -f broken-pod.yaml --dry-run=server
```
**This time you get a real error:**
```
The Pod "nginx-pod" is invalid: spec.containers[0].image: Required value
```

Word by word breakdown of this error message:
- `The Pod "nginx-pod" is invalid` — This Pod definition breaks Kubernetes' rules.
- `spec.containers[0].image` — This tells you exactly WHERE the problem is: inside `spec`, inside the `containers` list, item number `0` (the first item, since counting starts at 0), in the field called `image`.
- `Required value` — This field must have a value, and it doesn't.

**Answer to the task's question ("What error does Kubernetes give when the image field is missing?"):**
```
The Pod "nginx-pod" is invalid: spec.containers[0].image: Required value
```

## Step 6 (Extra): Try adding a field that doesn't exist at all

Edit `broken-pod.yaml` again, add back the image, and add a made-up field:
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:latest
    randomField: hello
    ports:
    - containerPort: 80
```

Run `--dry-run=client` again — again, no error is shown (client-side checking is too basic to catch this).

Run `--dry-run=server`:
```bash
kubectl apply -f broken-pod.yaml --dry-run=server
```
This time you get a different kind of error, ending with:
```
strict decoding error: unknown field "spec.containers[0].randomField"
```
This means: "I found a field called `randomField` that doesn't exist in Kubernetes' rules for a container — so I'm rejecting this."

## Summary Table

| Dry-run type | What it checks | Catches missing "image" field? | Catches made-up fields? |
|---|---|---|---|
| `--dry-run=client` | Basic YAML syntax only, on your own computer | No | No |
| `--dry-run=server` | Full, real checking by the actual Kubernetes API server | Yes | Yes |

**Lesson:** Always use `--dry-run=server` before applying anything important. `--dry-run=client` alone can give you false confidence that everything is fine.

### Task 4 Screenshots

**Missing "image" field error (Required value):**
![Task 4 - Missing Image Error](2026/day-51/Screenshots/day-51-task4-missing-image-error.png)

**Unknown/invalid field error (strict decoding error):**
![Task 4 - Unknown Field Error](2026/day-51/Screenshots/day-51-task4-unknown-field-error.png)

---

# TASK 5: Pod Labels and Filtering

**Goal:** Learn how labels work, and use them to group and filter Pods.

Remember, labels are simple tags (key-value pairs) you attach to resources. They don't affect how anything runs — they only exist so you can organize and select things later.

## Step 1: See every Pod's labels at once

```bash
kubectl get pods --show-labels
```
`--show-labels` adds an extra column to the output showing each Pod's labels.

## Step 2: Filter Pods by one label

```bash
kubectl get pods -l app=nginx
```
Word by word: `-l` stands for "label selector". This tells Kubernetes "only show me Pods that have this exact label". `app=nginx` means "the label called `app` must equal `nginx`".

This will only show `nginx-pod`, because that is the only Pod with that exact label.

Try another one:
```bash
kubectl get pods -l environment=dev
```
This will only show `busybox-pod`, since that's the only Pod with `environment: dev`.

## Step 3: Add a label to an already-running Pod (without editing any YAML file)

```bash
kubectl label pod nginx-pod environment=production
```
Word by word: `label` is the command to add/change/remove a label. `pod nginx-pod` says which Pod to change. `environment=production` is the new label to add.

Expected output: `pod/nginx-pod labeled`

## Step 4: Verify the label was added

```bash
kubectl get pods --show-labels
```
Now `nginx-pod` should show TWO labels: `app=nginx,environment=production`.

You can also add the SAME label to a different Pod:
```bash
kubectl label pod redis-pod environment=production
```

**Important experiment:** now run:
```bash
kubectl get pods -l environment=production
```
Both `nginx-pod` AND `redis-pod` will show up! This proves that a single label can group together completely unrelated Pods, even if their names and images are totally different.

## Step 5: Remove a label

```bash
kubectl label pod nginx-pod environment-
```
Word by word: Notice the dash (`-`) right after the label name, with no value. This special syntax means "remove this label" (instead of adding one).

Expected output: `pod/nginx-pod unlabeled`

## Step 6: Write a third manifest with at least 3 labels

Create a new file:
```bash
nano webapp-pod.yaml
```
Content:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
  labels:
    app: webapp
    environment: staging
    team: backend
spec:
  containers:
  - name: webapp
    image: nginx:latest
    ports:
    - containerPort: 80
```
This has THREE labels: `app`, `environment`, and `team` — exactly what the task asked for.

Save the file, then apply it:
```bash
kubectl apply -f webapp-pod.yaml
```
Expected output: `pod/webapp-pod created`

## Step 7: Practice filtering with the new Pod

Filter by team:
```bash
kubectl get pods -l team=backend
```
Only `webapp-pod` will show up.

Filter by TWO labels at once (using a comma):
```bash
kubectl get pods -l app=webapp,environment=staging
```
Word by word: the comma (`,`) between two conditions means **AND** — "only show Pods that match BOTH conditions at the same time". Only `webapp-pod` matches both, so only it shows up.

### Task 5 Screenshots

**Showing labels, filtering by single label, adding a label (part 1):**
![Task 5 - Labels and Filtering Part 1](2026/day-51/Screenshots/day-51-task5-labels-filtering-part1.png)

**Removing a label, verifying, creating webapp-pod.yaml, applying it (part 2):**
![Task 5 - Webapp Pod Creation Part 2](2026/day-51/Screenshots/day-51-task5-webapp-pod-creation-part2.png)

**Filtering webapp-pod by team and by multiple labels together (part 3):**
![Task 5 - Webapp Filtering Part 3](2026/day-51/Screenshots/day-51-task5-webapp-filtering-part3.png)

---

# TASK 6: Clean Up

**Goal:** Delete everything you created today, and understand an important limitation of plain Pods.

## Step 1: Delete Pods one by one, by name

```bash
kubectl delete pod nginx-pod
kubectl delete pod busybox-pod
kubectl delete pod redis-pod
```
Word by word: `delete` removes a resource. `pod nginx-pod` means "the Pod named nginx-pod". Each command deletes one Pod.

Expected output for each: `pod "<name>" deleted from default namespace`

## Step 2: Delete a Pod using its manifest file instead of its name

```bash
kubectl delete -f webapp-pod.yaml
```
Instead of typing the Pod's name, you can point to the file that describes it. Kubernetes reads the `kind` and `name` from inside the file and deletes that exact resource. Both methods (by name, or by file) do the same thing.

## Step 3: Verify everything is gone

```bash
kubectl get pods
```
Expected output:
```
No resources found in default namespace.
```
This confirms every Pod you created today has been removed.

## The Most Important Lesson of Today

When you delete a standalone Pod (or if it crashes, or the server it was running on fails), **it is gone forever**. There is no "controller" watching over it that will automatically bring it back.

This is exactly why, in real production systems, nobody runs bare/standalone Pods directly. Instead, they use a **Deployment** (which you will learn on Day 52). A Deployment constantly watches "how many copies (replicas) of this Pod should be running" and automatically creates a new one if any copy disappears. This automatic recovery behavior is called **self-healing**.

### Task 6 Screenshot

**All pods deleted, cluster verified empty:**
![Task 6 - Cleanup All Pods Deleted](2026/day-51/Screenshots/day-51-task6-cleanup-all-pods-deleted.png)

---

# Full Command Cheat-Sheet (Everything Used Today)

| Command | What it does |
|---|---|
| `kubectl apply -f file.yaml` | Create or update a resource based on a YAML file |
| `kubectl get pods` | List all Pods |
| `kubectl get pods -o wide` | List Pods with extra detail (IP, Node) |
| `kubectl get pods --show-labels` | List Pods along with their labels |
| `kubectl get pods -l key=value` | List only Pods matching a specific label |
| `kubectl get pods -l key1=val1,key2=val2` | List only Pods matching BOTH labels (AND) |
| `kubectl describe pod <name>` | Show full detailed info and event history for one Pod |
| `kubectl logs <name>` | Show the printed output/log history of a Pod's container |
| `kubectl exec -it <name> -- /bin/bash` | Open a shell session inside a running container |
| `kubectl label pod <name> key=value` | Add (or update) a label on an existing Pod |
| `kubectl label pod <name> key-` | Remove a label from an existing Pod |
| `kubectl run <name> --image=<image>` | Create a Pod instantly, without any YAML file (imperative) |
| `kubectl get pod <name> -o yaml` | Show the full, real YAML of a resource as it exists in the cluster |
| `kubectl run <name> --image=<image> --dry-run=client -o yaml` | Preview/generate a YAML template without creating anything |
| `kubectl apply -f file.yaml --dry-run=client` | Check basic YAML syntax only (no deep validation) |
| `kubectl apply -f file.yaml --dry-run=server` | Full validation by the real API server, without saving anything |
| `kubectl delete pod <name>` | Delete a Pod by name |
| `kubectl delete -f file.yaml` | Delete a Pod (or other resource) using its manifest file |

---

# Overall Takeaways From Today

1. Every Pod manifest needs 4 things: `apiVersion`, `kind`, `metadata`, `spec`.
2. `containerPort` in a manifest is just documentation — it does NOT expose a port to the outside world by itself.
3. The `command` field overrides what a container runs when it starts. This is essential for images like BusyBox that would otherwise exit immediately.
4. **Declarative** (write YAML + `apply`) is reusable and trackable — good for real projects. **Imperative** (`kubectl run`) is fast for quick testing but nothing is saved anywhere.
5. `kubectl get pod <name> -o yaml` shows the FULL real state (your input + Kubernetes' defaults + live status) — not just what you typed.
6. `--dry-run=client -o yaml` is a great shortcut for quickly generating a starting YAML template.
7. `--dry-run=client` only checks basic syntax. `--dry-run=server` does full, real validation. Always prefer `--dry-run=server` before applying anything that matters.
8. Labels let you group and filter resources, even completely unrelated ones — this is the foundation for how Services and Deployments will target Pods later.
9. Standalone Pods have NO self-healing. If deleted or lost, they are gone forever with nothing to recreate them. This is why Deployments exist for real production use
