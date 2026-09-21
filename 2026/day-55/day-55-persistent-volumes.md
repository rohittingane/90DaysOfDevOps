# Day 55 – Persistent Volumes (PV) and Persistent Volume Claims (PVC)

## Introduction

Containers are **ephemeral**. This is a fancy word that just means "temporary" or "short-lived." When a Pod dies or restarts, everything stored inside its container disappears — like writing on a whiteboard and then someone wipes it clean.

This is a **big problem** for anything that needs to remember data — databases, user uploads, logs, etc. If your database Pod restarts and loses all its data every time, that's a disaster.

**Today's mission:** Learn how Kubernetes solves this problem using **Persistent Volumes (PV)** and **Persistent Volume Claims (PVC)** — special objects that let data survive even when a Pod is deleted or recreated.

By the end of this guide, you will have:
- Personally seen data get destroyed (Task 1)
- Created real persistent storage that survives Pod deletion (Tasks 2–4)
- Understood how Kubernetes can auto-create storage for you (Tasks 5–6)
- Cleaned everything up safely (Task 7)

You do not need any prior knowledge of PV/PVC to follow this. Just a working Kubernetes cluster and `kubectl` installed.

---

## Task 1: See the Problem — Data Lost on Pod Deletion

### What are we doing?
We will create a Pod that writes the current date/time into a file. Then we'll delete that Pod, recreate it, and check if the old data is still there.

### Why are we doing this?
Before learning the solution, it's important to **feel the pain of the problem**. This proves — with our own eyes — that normal container storage does NOT survive a Pod restart.

### How does it work?
We use a Kubernetes volume type called `emptyDir`. Think of it as a **temporary empty folder** that:
- Gets created fresh when a Pod starts
- Lives only as long as that Pod is alive
- Is destroyed completely when the Pod is deleted

#### Step 1: Write the Pod manifest

Create a file named `emptydir-demo-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "date > /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: demo-volume
      mountPath: /data
  volumes:
  - name: demo-volume
    emptyDir: {}
```

**What does each line mean?**
- `kind: Pod` — we are creating a single Pod (a running container).
- `image: busybox` — a very small Linux image with basic commands, perfect for testing.
- `command: ["sh", "-c", "date > /data/message.txt && sleep 3600"]` — this writes the current date/time into a file, then sleeps for an hour so the Pod stays alive for us to check.
- `volumeMounts` — connects the container's `/data` folder to a volume named `demo-volume`.
- `volumes: emptyDir: {}` — defines that volume as an `emptyDir` (a temporary, empty folder tied to the Pod's lifetime).

#### Step 2: Apply the Pod

```bash
kubectl apply -f emptydir-demo-pod.yaml   # Create the Pod from our YAML file
```

Check that it's running:

```bash
kubectl get pods   # List all pods and see their status (should say "Running")
```

#### Step 3: Check the file exists

```bash
kubectl exec emptydir-demo -- cat /data/message.txt
# kubectl exec = "go inside this Pod and run a command"
# cat /data/message.txt = print the content of that file on screen
```

This shows the timestamp written when the Pod started. Note it down.

#### Step 4: Delete the Pod

```bash
kubectl delete pod emptydir-demo   # Completely remove this Pod (and its emptyDir with it)
```

#### Step 5: Recreate the same Pod

```bash
kubectl apply -f emptydir-demo-pod.yaml   # Same file, brand-new Pod + brand-new empty volume
```

#### Step 6: Check the file again

```bash
kubectl exec emptydir-demo -- cat /data/message.txt   # Check if old data is still there
```

### Verify: Is the timestamp the same or different?

**Answer: Different.** The first Pod wrote one timestamp. After deleting and recreating the Pod, a brand-new `emptyDir` was created, and the new Pod wrote a completely new timestamp. The old data is gone forever.

**Why did this happen?** Because `emptyDir` storage is tied to the Pod itself, not to the underlying disk in any permanent way. When the Pod dies, Kubernetes throws away that temporary folder.

**Output:**

![Pod created with emptyDir](2026/day-55/Screenshots/ss1-pod-create-emptydir.png)

![Data loss proof - timestamps are different](2026/day-55/Screenshots/ss2-data-loss-proof.png)

---

## Task 2: Create a PersistentVolume (Static Provisioning)

### What are we doing?
We will create a **PersistentVolume (PV)** — a piece of real, durable storage that exists independently of any Pod.

### Why are we doing this?
A PV solves the Task 1 problem. Its data does not disappear when a Pod is deleted, because the PV is a separate Kubernetes object with its own lifecycle.

### How does it work?

Think of it like this:
- **PV** = a plot of land / a house that already exists, built by the admin
- **PVC** (coming in Task 3) = a booking/reservation for that house
- **Pod** = the tenant who actually lives there

Today we are doing **"Static Provisioning"** — meaning a human (us, acting as the admin) manually writes the PV's specification, instead of Kubernetes automatically creating it.

#### Step 1: Write the PV manifest

Create a file named `pv-hostpath.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: ""              # No storage class - keeps this PV fully manual
  capacity:
    storage: 1Gi                    # This PV provides 1 Gigabyte of storage
  accessModes:
    - ReadWriteOnce                 # Only one node can read+write at a time
  persistentVolumeReclaimPolicy: Retain
    # When the PVC using this PV is deleted, KEEP the data (don't auto-delete)
  hostPath:
    path: /tmp/k8s-pv-data          # Folder on the Node's disk where data is actually stored
```

**What does each field mean?**
- `capacity: storage: 1Gi` — how much storage this PV offers (1 Gigabyte).
- `accessModes: ReadWriteOnce (RWO)` — only one Node can mount this volume for both read and write at the same time.
- `persistentVolumeReclaimPolicy: Retain` — this is important! When the claim (PVC) using this PV gets deleted, Kubernetes will **not** auto-delete the data. It stays safe until an admin manually removes it.
- `hostPath: path: /tmp/k8s-pv-data` — the real storage location: a folder on the Node's local disk.
- `storageClassName: ""` — leaving this empty tells Kubernetes "this PV is not tied to any automatic storage system; match it directly and manually with a PVC."

> **Important note:** `hostPath` is only good for learning/testing. In production, never use it, because it ties your data to one specific physical Node — if your Pod moves to a different Node, the data won't be there.

### Access Modes to know

| Mode | How many Nodes can use it? | What can they do? |
|---|---|---|
| **ReadWriteOnce (RWO)** | Only 1 Node at a time | Read + Write |
| **ReadOnlyMany (ROX)** | Many Nodes at once | Read only |
| **ReadWriteMany (RWX)** | Many Nodes at once | Read + Write |

**Simple analogy:**
- **RWO** — like your personal diary. Only you can write in it and read it at a time.
- **ROX** — like photocopies of a library book handed to everyone — they can all read, but nobody can write in it.
- **RWX** — like a shared Google Doc — everyone can read and write at the same time.

We used **RWO** here because `hostPath` storage lives on one specific Node's disk — only that Node can physically access it.

#### Step 2: Apply the PV

```bash
kubectl apply -f pv-hostpath.yaml   # Create the PersistentVolume from our YAML file
```

#### Step 3: Check the PV status

```bash
kubectl get pv   # List all PersistentVolumes and see their STATUS
```

### Verify: What is the STATUS of the PV?

**Answer: `Available`**

This means the PV was created successfully and is waiting to be claimed by a PVC. Nobody is using it yet — like an empty house waiting for a tenant.

**Output:**

![PV created with status Available](2026/day-55/Screenshots/ss3-pv-created-available.png)

---

## Task 3: Create a PersistentVolumeClaim

### What are we doing?
We will create a **PersistentVolumeClaim (PVC)** — a request for storage — and see it get matched (bound) to the PV we made in Task 2.

### Why are we doing this?
A Pod cannot use a PV directly. It must go through a PVC. The PVC is how a developer "asks" for storage without needing to know the low-level details (like the disk path) — the admin already handled that in the PV.

### How does it work?
Kubernetes automatically looks at all `Available` PVs and finds one that matches the PVC's request based on:
1. **Capacity** — the PVC's requested size must be ≤ the PV's size
2. **Access Mode** — must match

#### Step 1: Write the PVC manifest

Create a file named `pvc-demo.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: ""              # Must be empty too, to match our manual PV
  accessModes:
    - ReadWriteOnce                 # Must match the PV's access mode
  resources:
    requests:
      storage: 500Mi                # We are requesting 500 Megabytes
```

**What does each field mean?**
- `kind: PersistentVolumeClaim` — this is a *request* for storage, not the storage itself.
- `accessModes: ReadWriteOnce` — must match what the PV offers.
- `resources: requests: storage: 500Mi` — we're asking for 500 Megabytes, which is less than our PV's 1Gi, so it fits.
- `storageClassName: ""` — this MUST be empty (blank) to match our manually created PV. If you leave this field out completely, Kubernetes will try to use the cluster's *default* StorageClass instead, and your PVC will get stuck in `Pending` forever because it won't match a PV with no StorageClass.

#### Step 2: Apply the PVC

```bash
kubectl apply -f pvc-demo.yaml   # Create the PVC (the "request" for storage)
```

#### Step 3: Check both PVC and PV

```bash
kubectl get pvc   # See if our PVC got matched (bound) to a PV
kubectl get pv    # See if our PV's status changed from Available to Bound
```

### ⚠️ The Issue We Faced (and how we fixed it)

While doing this task ourselves, our PVC did **not** bind. Here's exactly what happened, why it happened, and how we solved it — because you will very likely hit the same issue.

**What we saw:**

```bash
kubectl get pvc
# NAME     STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
# my-pvc   Pending                                      standard       17s
#          ^^^^^^^ stuck in Pending instead of Bound

kubectl get pv
# NAME    STATUS      CLAIM
# my-pv   Available            <- our PV never got claimed, still just sitting there
```

**Why did this happen?**

When we first wrote `pvc-demo.yaml`, we forgot to add `storageClassName: ""`. Because that field was missing, Kubernetes automatically filled it in with the cluster's **default StorageClass**, which is called `standard` (we later confirmed this in Task 5).

Meanwhile, our PV (`my-pv`) had **no StorageClass at all** (an empty one), because we never gave it one either.

So we ended up with a mismatch:
- PVC was asking for a PV with `storageClassName: standard`
- PV actually had `storageClassName: ""` (blank)

They didn't match, so Kubernetes could never bind them — the PVC just sat there `Pending` forever.

**How we tried to fix it (and hit a second problem):**

Our first instinct was to just edit the PVC file and add the missing line, then re-apply:

```bash
nano pvc-demo.yaml        # Add storageClassName: "" to the file
kubectl apply -f pvc-demo.yaml   # Try to update the existing PVC
```

But this gave us a new error:

```
The PersistentVolumeClaim "my-pvc" is invalid: spec is immutable after creation
except resources.requests and volumeAttributesClassName for bound claims
```

**Why did THIS happen?** Once a PVC is created, Kubernetes treats most of its `spec` fields (including `storageClassName`) as **permanent** — you cannot change them afterward. Since our PVC already existed (with the wrong `standard` StorageClass baked in), simply editing the YAML and re-applying could not fix it. Kubernetes was refusing the update because it would require changing an immutable field.

**The actual fix:**

We had to completely **delete** the broken PVC and **recreate** it fresh, this time with the correct file from the start:

```bash
kubectl delete pvc my-pvc        # Remove the broken PVC entirely
kubectl apply -f pvc-demo.yaml   # Create a brand-new PVC with storageClassName: "" already correct
```

After this, checking again:

```bash
kubectl get pvc
# NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   AGE
# my-pvc   Bound    my-pv    1Gi        RWO            12s   <- fixed!

kubectl get pv
# NAME    STATUS   CLAIM
# my-pv   Bound    default/my-pvc   <- fixed!
```

**Lesson learned:** Always double check `storageClassName` matches on BOTH your PV and PVC *before* applying for the first time. If you get it wrong, don't try to edit and re-apply an existing PVC — delete it and create it again instead, since `storageClassName` cannot be changed after creation.

### Verify: What does the VOLUME column in `kubectl get pvc` show?

**Answer: `my-pv`**

The `VOLUME` column tells you exactly which PV this PVC is bound to. Our PVC (`my-pvc`, requesting 500Mi) successfully matched and bound to `my-pv` (which offers 1Gi).

**Note:** Even though we only requested 500Mi, the PVC's CAPACITY shows `1Gi`. This is because with static binding, a PVC gets the *entire* PV it binds to — Kubernetes does not "slice" a PV into smaller pieces.

Both objects now cross-reference each other:
- The PVC's `VOLUME` field shows `my-pv`
- The PV's `CLAIM` field shows `default/my-pvc`

**Output:**

![PVC created, initially Pending due to StorageClass mismatch](2026/day-55/Screenshots/ss4-pvc-created-pending.png)

![PVC and PV both Bound after fix](2026/day-55/Screenshots/ss5-pvc-pv-bound.png)

---

## Task 4: Use the PVC in a Pod — Data That Survives

### What are we doing?
We will create a Pod that uses our PVC (`my-pvc`) to store data, delete that Pod, recreate it, and prove the data survived — unlike in Task 1.

### Why are we doing this?
This is the actual payoff of everything we've built so far. We want to see, with our own eyes, that using a PVC means data is NOT lost when a Pod is deleted.

### How does it work?
Instead of `emptyDir`, our Pod's volume will point to `persistentVolumeClaim.claimName: my-pvc`. This connects the Pod's `/data` folder to the real, durable storage on disk (`/tmp/k8s-pv-data`), which exists independently of the Pod.

We will also use `>>` (append) instead of `>` (overwrite) in our command, so we can clearly see old and new data both stacking up in the same file.

#### Step 1: Write the Pod manifest

Create a file named `pod-with-pvc.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "echo Pod started at $(date) >> /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: my-storage
      mountPath: /data
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc              # This connects the Pod to our PVC from Task 3
```

**What's different here compared to Task 1?**
- We use `>>` (append) instead of `>` (overwrite) — so old lines in the file are NOT erased, new lines just get added below.
- Instead of `emptyDir: {}`, the volume now uses `persistentVolumeClaim: claimName: my-pvc` — pointing to our real persistent storage.

#### Step 2: Apply the Pod and check the file

```bash
kubectl apply -f pod-with-pvc.yaml                          # Create the Pod
kubectl exec pvc-demo-pod -- cat /data/message.txt          # Read the file - this is "Pod #1"
```

Note down this first timestamp.

#### Step 3: Delete the Pod

```bash
kubectl delete pod pvc-demo-pod   # Remove ONLY the Pod - the PVC and PV are untouched
```

Notice: we are only deleting the **Pod** here — not the PVC and not the PV. That's exactly why the data will survive.

#### Step 4: Recreate the same Pod

```bash
kubectl apply -f pod-with-pvc.yaml   # Create a NEW Pod, but it mounts the SAME PVC/PV as before
```

#### Step 5: Check the file again

```bash
kubectl exec pvc-demo-pod -- cat /data/message.txt   # This is "Pod #2" reading the same file
```

### Verify: Does the file contain data from both the first and second Pod?

**Answer: Yes.** The file shows two lines — one timestamp from the first Pod, and a second, later timestamp from the second (recreated) Pod. Both are visible together.

**Why did this work, when Task 1 failed?** Because the actual data lives on the PV's disk location (`/tmp/k8s-pv-data`), which is completely separate from the Pod. Deleting the Pod does not touch the PVC or the PV — so when a new Pod mounts the same PVC, it sees exactly the same data that was there before, and simply adds more to it.

**Output:**

![Pod using PVC, first entry written](2026/day-55/Screenshots/ss6-pod-pvc-first-entry.png)

![Data from both Pods present in the file after delete and recreate](2026/day-55/Screenshots/ss7-data-persisted-both-pods.png)

---

## Task 5: StorageClasses and Dynamic Provisioning

### What are we doing?
We will look at the **StorageClass** already present in our cluster and learn its settings, without creating anything yet.

### Why are we doing this?
So far, we manually created every PV ourselves. But imagine 100 developers each needing their own storage every day — an admin cannot manually create 100 PVs every day. **Dynamic Provisioning** solves this: developers only create a PVC, and Kubernetes automatically builds the PV behind the scenes using a StorageClass as the "blueprint."

### How does it work?

**Simple analogy:**
- **Static Provisioning** (what we did in Tasks 2–4) = you personally build a house, then someone moves in.
- **Dynamic Provisioning** = you hire a construction company (the StorageClass). A tenant (PVC) just says "I need a house," and the company automatically builds one on demand.

#### Step 1: List StorageClasses

```bash
kubectl get storageclass   # Short form: "kubectl get sc" - lists all StorageClasses in the cluster
```

#### Step 2: Get full details

```bash
kubectl describe storageclass   # Shows provisioner, reclaim policy, and volume binding mode in detail
```

### Understanding the output

| Field | Example Value | What it means |
|---|---|---|
| **PROVISIONER** | `rancher.io/local-path` | The actual "builder" that creates storage automatically. Different clusters use different provisioners (AWS uses `ebs.csi.aws.com`, Google uses `pd.csi.storage.gke.io`, etc). |
| **RECLAIMPOLICY** | `Delete` | Unlike our manual PV's `Retain` policy, this StorageClass automatically **deletes** the PV and its data when the matching PVC is deleted. |
| **VOLUMEBINDINGMODE** | `WaitForFirstConsumer` | The actual PV is NOT created immediately when the PVC is made. Kubernetes waits until a Pod actually tries to use that PVC, then creates the PV on the same Node the Pod is scheduled to, avoiding mismatches. |

### Verify: What is the default StorageClass in your cluster?

**Answer: `standard`** (using provisioner `rancher.io/local-path`)

You can tell it's the default because `kubectl get storageclass` shows `standard (default)` next to its name.

**Output:**

![StorageClass details showing provisioner, reclaim policy, and binding mode](2026/day-55/Screenshots/ss8-storageclass-details.png)

---

## Task 6: Dynamic Provisioning

### What are we doing?
We will create a PVC that uses the `standard` StorageClass — WITHOUT manually creating any PV — and watch Kubernetes automatically build one for us.

### Why are we doing this?
This is the real-world way most PVCs get created. Developers almost never manually create PVs in production — they just ask for storage via a PVC, and the cluster handles the rest.

### How does it work?

#### Step 1: Write the PVC manifest

Create a file named `dynamic-pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: standard          # Tells Kubernetes to auto-build a PV using this StorageClass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Mi
```

**What's different from Task 3's PVC?**
- `storageClassName: standard` instead of `""` — this tells Kubernetes "please automatically create a matching PV for me using the `standard` StorageClass."

#### Step 2: Apply it and check status

```bash
kubectl apply -f dynamic-pvc.yaml   # Create the PVC (no manual PV exists for it yet!)
kubectl get pvc                     # Check its status
kubectl get pv                      # Check if any new PV appeared (it won't, yet - see below)
```

**Important:** At this point, you'll likely see `dynamic-pvc` with STATUS `Pending`, and NO new PV yet in `kubectl get pv`. **This is expected, not an error!** Remember from Task 5 — our StorageClass uses `VolumeBindingMode: WaitForFirstConsumer`. This means the PV will only be created once an actual Pod tries to use this PVC.

#### Step 3: Write a Pod manifest that uses this PVC

Create a file named `pod-dynamic.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "echo Dynamic PV data at $(date) >> /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: my-storage
      mountPath: /data
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: dynamic-pvc          # Connects to our DYNAMIC PVC, not the manual one
```

#### Step 4: Apply the Pod and check everything again

```bash
kubectl apply -f pod-dynamic.yaml   # This is the "first consumer" - it triggers PV auto-creation
kubectl get pods                    # Confirm the Pod is Running
kubectl get pvc                     # dynamic-pvc should now say Bound
kubectl get pv                      # A brand-new, auto-named PV should now appear!
```

Now you should see:
- `dynamic-pvc` STATUS changes from `Pending` to `Bound`
- A brand-new PV appears in `kubectl get pv`, with an auto-generated name like `pvc-6dabda74-c492-401f-bf18-479671897469` — a name **we never typed ourselves**!

#### Step 5: Verify the data

```bash
kubectl exec dynamic-pod -- cat /data/message.txt   # Confirm the auto-created storage actually works
```

### Verify: How many PVs exist now? Which was manual, which was dynamic?

**Answer: 2 PVs exist.**

| PV Name | Type | How it was created |
|---|---|---|
| `my-pv` | **Manual (Static)** | We personally wrote `pv-hostpath.yaml` and applied it (Task 2) |
| `pvc-6dabda74-c492-...` | **Dynamic** | Kubernetes automatically created it the moment `dynamic-pod` tried to use `dynamic-pvc` |

You can also tell them apart by their `STORAGECLASS` and `RECLAIM POLICY` columns:
- Manual PV → `STORAGECLASS` is empty, `RECLAIM POLICY` is `Retain`
- Dynamic PV → `STORAGECLASS` is `standard`, `RECLAIM POLICY` is `Delete`

**Output:**

![Dynamic PVC created, initially Pending due to WaitForFirstConsumer](2026/day-55/Screenshots/ss9-dynamic-pvc-pending.png)

![Pod created using the dynamic PVC](2026/day-55/Screenshots/ss10-dynamic-pod-created.png)

![Both PVs visible, Bound status, and data verified](2026/day-55/Screenshots/ss11-dynamic-provisioning-proof.png)

---

## Task 7: Clean Up

### What are we doing?
We will delete everything we created, in the correct order, and observe the different behavior between our two PVs based on their reclaim policies.

### Why are we doing this?
Good DevOps practice means cleaning up test resources so they don't cost money or clutter the cluster. It also gives us one final, very clear demonstration of the difference between `Delete` and `Retain` reclaim policies.

### How does it work?

We must delete things in this order: **Pods first, then PVCs, then PVs.** If you delete a PVC while a Pod is still using it, the PVC can get stuck in a "Terminating" state.

#### Step 1: Delete all Pods

```bash
kubectl delete pod emptydir-demo    # Remove Pod from Task 1
kubectl delete pod pvc-demo-pod     # Remove Pod from Task 4
kubectl delete pod dynamic-pod      # Remove Pod from Task 6
kubectl get pods                    # Confirm all pods are gone
```

The last command should show `No resources found`.

#### Step 2: Delete both PVCs

```bash
kubectl delete pvc my-pvc         # Remove the manual PVC (linked to my-pv, policy = Retain)
kubectl delete pvc dynamic-pvc    # Remove the dynamic PVC (linked to auto PV, policy = Delete)
```

#### Step 3: Check what happened to the PVs

```bash
kubectl get pv   # Watch closely - the two PVs will behave very differently!
```

Here's what you'll see:
- The **dynamic PV** (`pvc-6dabda74-...`) is **completely gone** from the list — because its `RECLAIM POLICY` was `Delete`. Kubernetes automatically destroyed it along with the PVC.
- The **manual PV** (`my-pv`) is **still there**, but its STATUS changed from `Bound` to `Released` — because its `RECLAIM POLICY` was `Retain`. Kubernetes protects this data and will not reuse or delete it automatically.

**What does "Released" mean exactly?** It means this PV used to be bound to a PVC, but that PVC is now gone. The data is still safely sitting on disk. However, Kubernetes will not automatically hand this PV to a new PVC — an admin must manually clean it up or reset it before it can be reused.

#### Step 4: Manually delete the remaining PV

```bash
kubectl delete pv my-pv   # Since Retain doesn't auto-delete, WE must delete it ourselves
kubectl get pv            # Final check - cluster should now be fully clean
```

The final output should show `No resources found` — everything is now clean.

### Verify: Which PV was auto-deleted and which was retained? Why?

| PV | Reclaim Policy | What happened | Why |
|---|---|---|---|
| `pvc-6dabda74-...` (dynamic) | `Delete` | Auto-deleted completely | The `standard` StorageClass sets `Delete` as its default reclaim policy — when the PVC goes, the PV and its data go too |
| `my-pv` (manual) | `Retain` | Stayed behind as `Released` | We deliberately set `Retain` in Task 2 to protect the data — it only disappears when an admin explicitly deletes it |

**Output:**

![Full cleanup - pods deleted, PVCs deleted, dynamic PV gone, manual PV Released, then manually deleted](2026/day-55/Screenshots/ss12-cleanup-final.png)

---

## Final Summary

| Task | What we learned |
|---|---|
| 1 | `emptyDir` storage disappears when a Pod is deleted — a real data-loss risk |
| 2 | A PersistentVolume (PV) is real storage created independently of any Pod |
| 3 | A PersistentVolumeClaim (PVC) is a request that gets matched (bound) to a PV |
| 4 | Data written through a PVC survives Pod deletion and recreation |
| 5 | A StorageClass is a blueprint that tells Kubernetes how to auto-create storage |
| 6 | Dynamic Provisioning lets developers get storage with just a PVC — no manual PV needed |
| 7 | Reclaim Policy (`Retain` vs `Delete`) decides whether your data survives after cleanup |

**Key takeaway:** Always think carefully about your reclaim policy. `Retain` protects important data (like databases) from accidental deletion, while `Delete` is convenient for temporary or easily-recreated storage.
