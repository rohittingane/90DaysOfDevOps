# Day 60 – Capstone: Deploy WordPress + MySQL on Kubernetes

## About this challenge

This is the final day of a 10-day Kubernetes learning series (Day 51 to Day 60). Over those 10 days, the following concepts were covered one at a time: clusters, Pods, Deployments, Services, ConfigMaps, Secrets, storage, StatefulSets, resource management, autoscaling, and Helm.

Today's capstone task brings **all of those concepts together** into one real, working application: a WordPress website backed by a MySQL database, running entirely on Kubernetes.

You do **not** need to have followed Day 51–59 to understand this file. Every concept is explained here from scratch, in simple English, with the reasoning behind each step (not just the command to type). If you can run `kubectl` commands against a cluster, you can follow this guide from zero.

### What you need before starting
- A working Kubernetes cluster (this guide uses a **Kind** cluster — Kubernetes running inside Docker — but the steps work the same on Minikube or any other cluster, except where noted).
- `kubectl` installed and configured to talk to your cluster.
- `helm` installed (only needed for the bonus Task 7).
- Basic comfort with a Linux terminal (nano, cat, etc.).

### Expected final output
- A complete WordPress + MySQL stack running inside a `capstone` namespace.
- Proof that the app **self-heals** (survives pod crashes) and **persists data** (survives restarts without losing information).
- A markdown file (this one) documenting every step.
- Screenshots of the running WordPress site and `kubectl get all -n capstone`.

---

## Task 1: Create the Namespace

### What is a namespace?
A **namespace** is a way to create a separate, isolated "folder" inside a Kubernetes cluster. All the resources you create (Pods, Services, Secrets, etc.) can be grouped inside one namespace, keeping them separate from anything else running in the same cluster.

### Why use one for this project?
- It keeps the WordPress + MySQL stack cleanly separated from any other testing or projects in the cluster.
- In real companies, teams almost always use a separate namespace per project or environment (dev, staging, prod) to avoid accidentally mixing things up or deleting the wrong resource.

### Step 1.1 — Create the namespace
```bash
kubectl create namespace capstone
```
- `kubectl create namespace` is the command used to create a new namespace.
- `capstone` is simply the name we are giving it.

**Verify it worked:**
```bash
kubectl get namespaces
```
You should see `capstone` in the list with status `Active`.

### Step 1.2 — Make it your default namespace
```bash
kubectl config set-context --current --namespace=capstone
```
- Normally, every `kubectl` command silently targets the `default` namespace unless you add `-n <namespace>` to every single command.
- This command changes your **current context** so that `kubectl` automatically targets `capstone` from now on — saving you from typing `-n capstone` on every command for the rest of this guide.

**Verify it worked:**
```bash
kubectl config view --minify | grep namespace
```
You should see `namespace: capstone`.

📸 **Screenshot:**
![Task 1 - Namespace setup](2026/day-60/Screenshots/day-60-task1-namespace-setup.png)

---

## Task 2: Deploy MySQL (concepts from earlier days: Secrets, Headless Services, StatefulSets, Persistent Storage)

### Step 2.1 — Create a Secret for MySQL credentials

**What is a Secret?**
A Kubernetes **Secret** is an object built to hold sensitive data — passwords, tokens, keys — separately from your application code or plain configuration files.

**Why use one here?**
MySQL needs a root password, a database name, a username, and a password. These should never be hardcoded in plain YAML files that might get committed to Git. A Secret keeps this data managed by Kubernetes, and only Pods that are explicitly told to use it can read it.

**How — create the file** `mysql-secret.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: capstone
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: "RootPass123!"
  MYSQL_DATABASE: "wordpress"
  MYSQL_USER: "wpuser"
  MYSQL_PASSWORD: "WpUserPass123!"
```
- `type: Opaque` is the default/generic Secret type for arbitrary key-value data.
- `stringData` lets you write plain-text values directly — Kubernetes automatically base64-encodes them for you (as opposed to the `data` field, where you'd have to encode values yourself).
- `MYSQL_DATABASE: wordpress` matters because this is exactly the database name WordPress will later look for.

**Apply it:**
```bash
kubectl apply -f mysql-secret.yaml
```
Expected output: `secret/mysql-secret created`

**Verify:**
```bash
kubectl get secrets -n capstone
```
You should see `mysql-secret` of type `Opaque` with `DATA: 4` (because we gave it 4 key-value pairs).

### Step 2.2 — Create a Headless Service for MySQL

**What is a Headless Service?**
A normal Kubernetes Service gives you one single IP address that load-balances traffic across multiple Pods. A **Headless Service** (created by setting `clusterIP: None`) does **not** do that — instead, it lets you reach each individual Pod directly by its own unique DNS name.

**Why does MySQL need this instead of a normal Service?**
Because we are about to create MySQL as a **StatefulSet** (next step), and StatefulSet Pods need stable, individual network identities (e.g., `mysql-0`, `mysql-1`, ...). A Headless Service is what makes each Pod reachable by its own name, in the pattern:
```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

**How — create the file** `mysql-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: capstone
spec:
  clusterIP: None       # This is what makes it "headless"
  selector:
    app: mysql           # Targets any pod carrying this label
  ports:
    - port: 3306          # MySQL's default port
      targetPort: 3306
```

**Apply it:**
```bash
kubectl apply -f mysql-service.yaml
```
Expected output: `service/mysql created`

**Verify:**
```bash
kubectl get svc -n capstone
```
You should see `mysql` with `TYPE: ClusterIP` and `CLUSTER-IP: None` — the `None` is what confirms it's headless.

### Step 2.3 — Create a StatefulSet for MySQL

**What is a StatefulSet, and why not a normal Deployment?**
- A **Deployment** gives every Pod a random name and (usually) shared/rotating storage — fine for stateless apps.
- A **StatefulSet** gives every Pod a **fixed, predictable name** (`mysql-0`, `mysql-1`, ...) and its **own dedicated, permanent storage** that survives even if the Pod is deleted and recreated.
- Databases need both stable identity and permanent storage, so StatefulSet is the right tool for MySQL — never use a plain Deployment for a database in production.

**How — create the file** `mysql-statefulset.yaml`:
```yaml
apiVersion: apps/v1                    # API version for StatefulSet
kind: StatefulSet                      # We are creating a StatefulSet (for databases)
metadata:
  name: mysql                          # Name of this StatefulSet
  namespace: capstone                  # Deploy it inside "capstone" namespace
spec:
  serviceName: mysql                   # Links to our headless Service (must match its name)
  replicas: 1                          # Only 1 MySQL pod needed
  selector:
    matchLabels:
      app: mysql                       # Manages pods with this label
  template:
    metadata:
      labels:
        app: mysql                     # Pods get this label (must match selector above)
    spec:
      containers:
        - name: mysql                  # Name of the container
          image: mysql:8.0             # MySQL version 8.0 image
          envFrom:
            - secretRef:
                name: mysql-secret     # Load all values from our Secret as env vars
          ports:
            - containerPort: 3306      # MySQL listens on this port inside the container
          resources:
            requests:                  # Minimum guaranteed resources
              cpu: "250m"                # 0.25 CPU core
              memory: "512Mi"            # 512 MB RAM
            limits:                    # Maximum allowed resources
              cpu: "500m"                # 0.5 CPU core max
              memory: "1Gi"              # 1 GB RAM max
          volumeMounts:
            - name: mysql-data          # Must match volumeClaimTemplates name below
              mountPath: /var/lib/mysql # MySQL stores its actual data files here
  volumeClaimTemplates:                # Auto-creates a unique PVC (storage claim) per pod
    - metadata:
        name: mysql-data               # Must match volumeMounts name above
      spec:
        accessModes: ["ReadWriteOnce"] # Only one pod can mount this storage at a time
        resources:
          requests:
            storage: 1Gi                # Request 1 GB of persistent storage
```

**Key ideas explained:**
- `envFrom.secretRef` pulls **all four** values from the Secret in one go, as environment variables. The MySQL Docker image itself reads these variables at startup and automatically creates the root user, database, and app user for you.
- `resources.requests` = the minimum guaranteed CPU/memory the Pod needs to be scheduled; `resources.limits` = the hard ceiling it can never exceed, protecting the rest of the cluster.
- `volumeMounts` says "inside the container, put the storage at this path." `volumeClaimTemplates` is what actually requests and creates that storage. The `name` in both must match exactly, or they won't connect.
- Because this is a StatefulSet, each Pod (`mysql-0`) automatically gets its own PersistentVolumeClaim (named `mysql-data-mysql-0`) — so even if the Pod is deleted, the data survives and the new Pod reconnects to the same storage.

**Apply it:**
```bash
kubectl apply -f mysql-statefulset.yaml
```
Expected output: `statefulset.apps/mysql created`

**Verify the Pod came up:**
```bash
kubectl get pods -n capstone
```
Wait until you see `mysql-0` with `READY: 1/1` and `STATUS: Running`.

**Verify the storage was created:**
```bash
kubectl get pvc -n capstone
```
You should see a PVC named `mysql-data-mysql-0` with `STATUS: Bound` and `CAPACITY: 1Gi`.

### Step 2.4 — Verify MySQL actually works
```bash
kubectl exec -it mysql-0 -n capstone -- mysql -u wpuser -pWpUserPass123! -e "SHOW DATABASES;"
```
- `kubectl exec -it mysql-0` runs a command **inside** the `mysql-0` Pod.
- We log in as the `wpuser` user with its password (use whatever values you actually put in your Secret).
- `-e "SHOW DATABASES;"` runs that SQL query immediately after logging in.
- You should see `wordpress` listed among the databases — this confirms MySQL correctly read the Secret and auto-created the database.

(Note: you'll see a harmless warning about using a password on the command line — that's just a security note, not an error.)

📸 **Screenshots:**
![Task 2 Part 1 - Secret and Service](2026/day-60/Screenshots/day-60-task2-part1-secret-service.png)
![Task 2 Part 2 - StatefulSet and verification](2026/day-60/Screenshots/day-60-task2-part2-statefulset-verify.png)

---

## Task 3: Deploy WordPress (concepts from earlier days: ConfigMaps, Deployments, envFrom/secretKeyRef, Probes)

### Step 3.1 — Create a ConfigMap for WordPress settings

**What is a ConfigMap, and how is it different from a Secret?**
A **ConfigMap** stores non-sensitive configuration data (things that aren't secret, like hostnames or database names). A **Secret** is for sensitive data (passwords, keys). Both can be injected into a Pod as environment variables.

**How — create the file** `wordpress-configmap.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config
  namespace: capstone
data:
  WORDPRESS_DB_HOST: "mysql-0.mysql.capstone.svc.cluster.local:3306"
  WORDPRESS_DB_NAME: "wordpress"
```
- `WORDPRESS_DB_HOST` uses the exact DNS naming pattern that our Headless Service (from Task 2) makes possible: `<pod>.<service>.<namespace>.svc.cluster.local`. This lets WordPress reach the MySQL Pod directly by name, rather than through a random IP.
- `WORDPRESS_DB_NAME` must match the database name we set in the MySQL Secret (`wordpress`).

**Apply it:**
```bash
kubectl apply -f wordpress-configmap.yaml
```
Expected output: `configmap/wordpress-config created`

**Verify:**
```bash
kubectl get configmap wordpress-config -n capstone -o yaml
```

### Step 3.2 — Create the WordPress Deployment

**Why a Deployment here, not a StatefulSet?**
WordPress itself is **stateless** — its actual data (posts, users) lives in MySQL, not in the WordPress Pod itself. So a normal Deployment (with multiple interchangeable replicas) is the right tool.

**How — create the file** `wordpress-deployment.yaml`:
```yaml
apiVersion: apps/v1                       # API version for Deployment
kind: Deployment                          # Creating a stateless Deployment
metadata:
  name: wordpress                         # Name of this Deployment
  namespace: capstone                     # Inside capstone namespace
spec:
  replicas: 2                             # Run 2 pods (basic high availability)
  selector:
    matchLabels:
      app: wordpress                      # Manages pods with this label
  template:
    metadata:
      labels:
        app: wordpress                    # Pods get this label
    spec:
      containers:
        - name: wordpress                       # Container name
          image: wordpress:latest               # WordPress image
          envFrom:
            - configMapRef:
                name: wordpress-config          # Import DB_HOST, DB_NAME from ConfigMap
          env:
            - name: WORDPRESS_DB_USER           # WordPress expects this exact env var name
              valueFrom:
                secretKeyRef:
                  name: mysql-secret            # Read from the MySQL secret
                  key: MYSQL_USER                # ...but pick just this one key
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_PASSWORD
          ports:
            - containerPort: 80                 # Apache/WordPress listens here
          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"
            limits:
              cpu: "300m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /wp-login.php               # Is the app alive?
              port: 80
            initialDelaySeconds: 30              # Wait 30s before the first check
            periodSeconds: 10                     # Then check every 10s
          readinessProbe:
            httpGet:
              path: /wp-login.php               # Is the app ready to receive traffic?
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 5
```

**Key ideas explained:**
- `envFrom.configMapRef` imports **all** ConfigMap values at once, using their existing key names.
- `env` with `secretKeyRef` is used instead, because the key names inside the Secret (`MYSQL_USER`, `MYSQL_PASSWORD`) don't match what WordPress expects (`WORDPRESS_DB_USER`, `WORDPRESS_DB_PASSWORD`). This approach lets you rename a value while pulling it from a Secret.
- **Liveness probe**: if this check fails, Kubernetes assumes the container is stuck/broken and **restarts** it automatically. This is the mechanism behind "self-healing."
- **Readiness probe**: if this check fails, Kubernetes temporarily stops sending traffic to that Pod (without restarting it) until it passes again — useful during slow startups.
- Both probes hit `/wp-login.php` because if that page loads successfully, it proves WordPress is not just alive, but also correctly talking to its database.

**Apply it:**
```bash
kubectl apply -f wordpress-deployment.yaml
```
Expected output: `deployment.apps/wordpress created`

**Wait for both Pods to become ready:**
```bash
kubectl get pods -n capstone -w
```
Wait until you see two `wordpress-...` Pods both showing `1/1 Running`, then press `Ctrl+C`.

📸 **Screenshots:**
![Task 3 Part 1 - ConfigMap](2026/day-60/Screenshots/day-60-task3-part1-configmap.png)
![Task 3 Part 2 - Deployment pods ready](2026/day-60/Screenshots/day-60-task3-part2-deployment-pods-ready.png)

---

## Task 4: Expose WordPress (concepts from earlier days: NodePort Services)

### Step 4.1 — Create a NodePort Service

**Why do we need this?**
So far, WordPress Pods are only reachable *from inside* the cluster. A **NodePort** Service opens a fixed port on every cluster node, mapping it to the Pods, so it becomes reachable from outside — like from your browser.

**How — create the file** `wordpress-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: capstone
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
    - port: 80                # Internal cluster port
      targetPort: 80           # The port WordPress container listens on
      nodePort: 30080           # Fixed external port
```

**Apply it:**
```bash
kubectl apply -f wordpress-service.yaml
```
Expected output: `service/wordpress created`

**Verify:**
```bash
kubectl get svc -n capstone
```
You should see `wordpress` as `TYPE: NodePort` with `PORT(S): 80:30080/TCP`.

### Step 4.2 — Access WordPress in your browser

How you access it depends on your cluster type:

**If using Minikube:**
```bash
minikube service wordpress -n capstone
```
This opens the browser for you automatically.

**If using Kind (as in this guide):**
Kind runs your cluster inside Docker, so its "nodes" are Docker containers, not real machines — this means a NodePort is **not** directly reachable from outside the host, even if firewall/security group ports are opened. Instead, use port-forwarding:
```bash
kubectl port-forward svc/wordpress 8080:80 -n capstone
```
This creates a tunnel: your local port `8080` → the cluster's WordPress Service on port `80`. Keep this terminal running (don't close it).

**If your cluster is on a remote server (e.g., an EC2 instance) and you're accessing it via SSH from your own laptop:**
The port-forward above only listens on `127.0.0.1` of the remote server by default, so you also need an SSH tunnel from your own laptop:
```bash
ssh -i your-key.pem -L 8080:localhost:8080 ubuntu@<server-public-ip>
```
Then, in your laptop's browser, go to:
```
http://localhost:8080
```

Alternatively, if you'd rather use the NodePort directly (works on real cloud VMs, not Kind), open port `30080` in your cloud provider's firewall/security group and browse to:
```
http://<server-public-ip>:30080
```

### Step 4.3 — Complete the WordPress setup wizard
1. Fill in Site Title, Username, Password, Email.
2. Click **Install WordPress**.
3. Log in with the username/password you just created.
4. In the Dashboard, go to **Posts → Add New**, write a title and some content, then click **Publish**.

📸 **Screenshots:**
![Task 4 - NodePort Service and port-forward](2026/day-60/Screenshots/day-60-task4-nodeport-service-portforward.png)
![Task 4 - Blog post draft](2026/day-60/Screenshots/day-60-task4-blogpost-draft.png)
![Task 4 - Blog post published](2026/day-60/Screenshots/day-60-task4-blogpost-published.png)

---

## Task 5: Test Self-Healing and Persistence

This is the most important test in the whole capstone — it proves the setup is production-grade, not just "working by luck."

### Step 5.1 — Delete a WordPress Pod
```bash
kubectl get pods -n capstone
kubectl delete pod <one-of-the-wordpress-pod-names> -n capstone
kubectl get pods -n capstone -w
```
**What to expect:** within seconds, the Deployment automatically creates a brand-new Pod (with a **new random name**, since it's a Deployment) to replace the deleted one, bringing the replica count back to 2. Refresh the WordPress site in your browser — it should still work (the other replica kept serving traffic the whole time).

### Step 5.2 — Delete the MySQL Pod
```bash
kubectl delete pod mysql-0 -n capstone
kubectl get pods -n capstone -w
```
**What to expect:** the StatefulSet recreates the Pod, and — unlike the WordPress case — it comes back with the **exact same name**, `mysql-0`. This is the core difference between Deployments and StatefulSets.

### Step 5.3 — Confirm your data survived
Refresh the WordPress site in your browser. Your blog post from Task 4 should **still be there**.

**Why this matters:** when `mysql-0` was deleted and recreated, it reconnected to the same PersistentVolumeClaim (`mysql-data-mysql-0`) created back in Task 2. If persistent storage wasn't set up correctly, the new MySQL Pod would have started with a completely empty, fresh database, and your blog post would have disappeared.

📸 **Screenshots:**
![Task 5 Part 1 - Pod delete terminal log](2026/day-60/Screenshots/day-60-task5-part1-pod-delete-terminal-log.png)
![Task 5 Part 2 - Blog post persisted](2026/day-60/Screenshots/day-60-task5-part2-blogpost-persisted.png)

---

## Task 6: Set Up HPA (Horizontal Pod Autoscaler)

### What is HPA?
**HPA** automatically increases or decreases the number of Pods in a Deployment based on real-time resource usage (commonly CPU), instead of you having to manually change the replica count.

### Prerequisite: Metrics Server
HPA needs a **Metrics Server** running in the cluster to read CPU usage data. Check if it's installed:
```bash
kubectl get deployment metrics-server -n kube-system
```
If it's missing, you'll need to install it before HPA can report real numbers (it will otherwise show `<unknown>` targets).

### Step 6.1 — Write the HPA manifest
Create the file `wordpress-hpa.yaml`:
```yaml
apiVersion: autoscaling/v2                 # API version for HPA
kind: HorizontalPodAutoscaler                # We are creating an HPA
metadata:
  name: wordpress-hpa
  namespace: capstone
spec:
  scaleTargetRef:                             # Which Deployment to watch/scale
    apiVersion: apps/v1
    kind: Deployment
    name: wordpress                           # Must match our WordPress Deployment's name
  minReplicas: 2                              # Never scale below 2 pods
  maxReplicas: 10                             # Never scale above 10 pods
  metrics:
    - type: Resource
      resource:
        name: cpu                             # Watch CPU usage
        target:
          type: Utilization
          averageUtilization: 50              # Target: keep average CPU around 50%
```

**Apply it:**
```bash
kubectl apply -f wordpress-hpa.yaml
```
Expected output: `horizontalpodautoscaler.autoscaling/wordpress-hpa created`

**Verify:**
```bash
kubectl get hpa -n capstone
```
You should see something like:
```
NAME            REFERENCE              TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
wordpress-hpa   Deployment/wordpress   cpu: 6%/50%   2         10        2          21s
```
- `TARGETS` shows current CPU % vs. the 50% target.
- `MINPODS`/`MAXPODS` confirm the boundaries.
- `REPLICAS` shows how many Pods currently exist (it will grow toward `MAXPODS` if CPU usage rises above 50%, and shrink back toward `MINPODS` if usage drops).

### Step 6.2 — See the complete picture
```bash
kubectl get all -n capstone
```
This lists every resource type together: Pods, Services, Deployment, ReplicaSet, StatefulSet, and the HPA — a full snapshot of everything built so far.

📸 **Screenshot:**
![Task 6 - HPA and complete get all](2026/day-60/Screenshots/day-60-task6-hpa-getall-final.png)

---

## Task 7 (Bonus): Compare with Helm

### What is Helm?
**Helm** is a package manager for Kubernetes. Instead of writing every YAML file by hand (as we did in Tasks 1–6), you can install a pre-built, ready-made "chart" with a single command, and it creates all the necessary resources automatically.

### Step 7.1 — Set up a separate namespace
Using a separate namespace keeps this comparison from disturbing anything we manually built earlier.
```bash
kubectl create namespace helm-demo
```

### Step 7.2 — Install WordPress via Helm
> **Important note (2026):** Bitnami discontinued its old `https://charts.bitnami.com/bitnami` Helm repository. Charts are now distributed via an **OCI registry** on Docker Hub instead. If `helm repo add`/`helm repo update` with the old URL hangs or fails, use the new OCI-based install method directly — no `helm repo add` needed:
```bash
helm install wp-helm oci://registry-1.docker.io/bitnamicharts/wordpress -n helm-demo
```
This single command pulls the chart and creates everything needed: WordPress Deployment, a MariaDB StatefulSet (Bitnami's chart uses MariaDB instead of MySQL by default), Services, Secrets, and ConfigMaps — automatically.

### Step 7.3 — Compare the two approaches
```bash
kubectl get all -n helm-demo
```

| | **Manual (Tasks 1–6)** | **Helm** |
|---|---|---|
| Pods | 3 (mysql-0, wordpress×2) | 2 (mariadb-0, wordpress×1, default replica count) |
| Services | 2 (headless MySQL, NodePort WordPress) | 3 (ClusterIP + headless MariaDB, LoadBalancer WordPress) |
| Deployment / StatefulSet | Hand-written by us | Auto-generated by the chart |
| HPA | We added it ourselves | Not included by default |
| Time to deploy | Much longer (wrote and understood every file) | One command, seconds |
| Control | Full control over every setting | Controlled via chart defaults / `values.yaml` overrides |
| Understanding gained | High — you see exactly how every piece connects | Lower — internal wiring is hidden inside chart templates |

**Takeaway:** Helm is faster and great for production use with well-tested charts, but writing manifests by hand (like we did) is how you actually learn what's happening under the hood — and gives full control for custom, specific requirements.

### Step 7.4 — Clean up the Helm deployment
```bash
helm uninstall wp-helm -n helm-demo
kubectl delete namespace helm-demo
```
- `helm uninstall` removes every resource that the Helm release created, in one command.
- Deleting the namespace afterward removes anything left over (like the namespace object itself).

📸 **Screenshot:**
![Task 7 - Helm comparison](2026/day-60/Screenshots/day-60-task7-helm-getall-comparison.png)

---

## Task 8: Clean Up and Reflect

### Step 8.1 — Take a final look before deleting anything
```bash
kubectl get all -n capstone
```
This is your "before" snapshot — useful to compare against after deletion.

### Step 8.2 — Count the concepts you used
This single capstone deployment touched **12 different Kubernetes concepts**, all learned individually across the past 10 days and combined here into one working system:

| # | Concept | Where it was used |
|---|---|---|
| 1 | Namespace | `capstone` — kept everything isolated |
| 2 | Secret | `mysql-secret` — hid the password and DB credentials |
| 3 | ConfigMap | `wordpress-config` — held non-sensitive DB host/name |
| 4 | PVC (`volumeClaimTemplates`) | MySQL's 1Gi persistent storage |
| 5 | StatefulSet | MySQL — fixed Pod name + stable storage |
| 6 | Headless Service | `mysql` — direct DNS access to the Pod |
| 7 | Deployment | WordPress — 2 stateless replicas |
| 8 | NodePort Service | `wordpress` — external browser access |
| 9 | Resource requests/limits | Applied to both MySQL and WordPress containers |
| 10 | Probes (liveness + readiness) | WordPress — self-healing + traffic control |
| 11 | HPA | `wordpress-hpa` — CPU-based automatic scaling |
| 12 | Helm | Bonus Task 7 — packaged install comparison |

### Step 8.3 — Delete the namespace
```bash
kubectl delete namespace capstone
```
**Why this works so cleanly:** deleting a namespace tells Kubernetes to cascade-delete **everything** inside it — every Pod, Service, Deployment, StatefulSet, Secret, ConfigMap, PVC, and HPA — automatically, in one command. This is one of the biggest advantages of grouping related resources into a namespace.

**Verify everything is really gone:**
```bash
kubectl get all -n capstone
```
Expected output: `No resources found in capstone namespace.`

⚠️ **Important warning for real-world use:** deleting a namespace also deletes its PersistentVolumeClaims — meaning any real data stored there is gone permanently. This is fine for a learning capstone, but in production this action requires extreme caution.

### Step 8.4 — Reset your default namespace
```bash
kubectl config set-context --current --namespace=default
```
Since `capstone` no longer exists, this points your terminal's default namespace back to `default`, keeping your environment clean for whatever you work on next.

📸 **Screenshots:**
![Task 8 - Before delete final state](2026/day-60/Screenshots/day-60-task8-before-delete-final-state.png)
![Task 8 - Cleanup and verify](2026/day-60/Screenshots/day-60-task8-cleanup-and-verify.png)

---

## Final Summary

By completing this capstone, a full WordPress + MySQL production-style application was deployed on Kubernetes using nothing but hand-written YAML manifests (plus one Helm comparison for contrast). Every core Kubernetes building block — namespaces, secrets, config, storage, stateful and stateless workloads, networking, scaling, and packaging — was used together in a single, working system, and each behavior (self-healing, persistence, autoscaling, cleanup) was actually tested and verified, not just assumed.
