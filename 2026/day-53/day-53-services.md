# Day 53 – Kubernetes Services

This document explains everything I did on Day 53 of my DevOps challenge, in simple English, so that anyone (even someone who missed the task) can read this file and follow along step by step.

## What is the problem we are solving?

When you run an application in Kubernetes, it runs inside **Pods**. But Pods have two problems:

1. **Pod IPs are not stable** – if a Pod restarts or gets replaced, it gets a brand-new IP address.
2. **A Deployment runs multiple Pods** – so which IP address should a client actually connect to?

A **Service** solves both problems. It gives your Pods:

- A **stable IP address and DNS name** that never changes, even if the Pods behind it restart.
- **Load balancing** – it automatically spreads traffic across all matching Pods.

```
[Client] --> [Service (stable IP)] --> [Pod 1]
                                   --> [Pod 2]
                                   --> [Pod 3]
```

There are 3 main types of Services, and today I created all 3 and tested them.

---

## Task 1: Deploy the Application

First, I created a Deployment. This is the application that the Services will later expose.

**File: `app-deployment.yaml`**

```yaml
apiVersion: apps/v1          # Tells Kubernetes this is a "Deployment" object
kind: Deployment              # Type of resource = Deployment
metadata:
  name: web-app                # Name of this Deployment
  labels:
    app: web-app                 # A tag used to identify this Deployment
spec:
  replicas: 3                   # Run 3 copies (Pods) of the app
  selector:
    matchLabels:
      app: web-app                 # This Deployment only manages Pods with this label
  template:                     # This is the "blueprint" used to create each Pod
    metadata:
      labels:
        app: web-app                # Every Pod created gets this exact label
                                     # (Services later use this same label to find these Pods)
    spec:
      containers:
      - name: nginx                # Name of the container inside the Pod
        image: nginx:1.25            # Which Docker image to run
        ports:
        - containerPort: 80           # The container listens on port 80 (Nginx's default port)
```

**Commands used:**

```bash
kubectl apply -f app-deployment.yaml
# This command tells Kubernetes: "read this file and create/update whatever is described in it"

kubectl get pods -o wide
# This lists all Pods, and -o wide adds extra columns like Pod IP and which Node it is running on
```

**What I saw:** All 3 Pods came up as `Running`, each with its own IP address (for example `10.244.0.5`, `10.244.0.6`, `10.244.0.7`). These IPs will change if the Pods restart — this is exactly the problem that Services are built to solve.

**Screenshot:**

![Deployment and Pods Running](2026/day-53/Screenshots/day53-1-deployment-pods-running.png)

---

## Task 2: ClusterIP Service (Internal Access)

`ClusterIP` is the **default** Service type. It gives the Pods a stable address, but this address can only be reached **from inside the cluster** — not from the outside world.

**File: `clusterip-service.yaml`**

```yaml
apiVersion: v1                    # Services use the "v1" core API
kind: Service                      # Type of resource = Service (not Deployment, not Pod)
metadata:
  name: web-app-clusterip           # Name of this Service
                                     # This name is also usable as a DNS name inside the cluster
spec:
  type: ClusterIP                   # ClusterIP = only reachable from inside the cluster
                                      # (this is also the default, so it's optional to write)
  selector:
    app: web-app                     # This is how the Service finds its Pods:
                                       # it looks for any Pod with label "app: web-app"
                                       # -> this matches all 3 Nginx Pods from Task 1
  ports:
  - port: 80                          # The port the SERVICE itself listens on
    targetPort: 80                     # The port ON THE POD that traffic gets forwarded to
```

**Commands used:**

```bash
kubectl apply -f clusterip-service.yaml
# Creates the Service from the file above

kubectl get services
# Lists all Services and shows their CLUSTER-IP, type, and ports
```

**What I saw:** A new Service called `web-app-clusterip` appeared with a stable `CLUSTER-IP` (in my case `10.96.239.2`). This IP will not change even if the Pods behind it restart.

### Testing the Service from inside the cluster

To prove the Service actually works, I created a temporary test Pod and used it to send a request to the Service:

```bash
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh
# kubectl run          -> creates a new temporary Pod
# --image=busybox      -> uses a small Linux image with basic networking tools
# --rm                 -> automatically deletes this Pod once we exit
# -it                  -> gives us an interactive terminal inside the Pod
# --restart=Never      -> don't recreate this Pod if it stops; it's just for testing
# -- sh                -> once inside, start a shell
```

Once inside the test Pod's shell, I ran:

```bash
wget -qO- http://web-app-clusterip
# wget      -> tool to fetch a web page
# -q        -> quiet mode, hides extra logs
# -O-       -> print the result on the screen instead of saving it to a file
# web-app-clusterip -> the Service name; Kubernetes automatically turns this into an IP
```

**What I saw:** The Nginx welcome page (`Welcome to nginx!`) was returned. This proves the Service successfully forwarded my request to one of the 3 Pods, without me ever needing to know a Pod IP.

Then I exited the test Pod:

```bash
exit
# Leaves the shell; since we used --rm, the test Pod is automatically deleted
```

**Screenshots:**

![ClusterIP YAML Explained](2026/day-53/Screenshots/day53-3-clusterip-service-yaml-explained.png)
![ClusterIP wget Success](2026/day-53/Screenshots/day53-4-clusterip-wget-success.png)
![ClusterIP DNS Test and Exit](2026/day-53/Screenshots/day53-5-clusterip-dns-test-exit.png)

---

## Task 3: Discover Services with DNS

Kubernetes has a **built-in DNS server**. Every Service automatically gets a DNS name in this format:

```
<service-name>.<namespace>.svc.cluster.local
```

So my ClusterIP Service can be reached using either:
- The **short name**: `web-app-clusterip` (works only within the same namespace)
- The **full DNS name**: `web-app-clusterip.default.svc.cluster.local` (works across namespaces too)

**Commands used (inside a new test Pod):**

```bash
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh
# Same as before: creates a temporary test Pod with an interactive shell

wget -qO- http://web-app-clusterip
# Testing the SHORT name

wget -qO- http://web-app-clusterip.default.svc.cluster.local
# Testing the FULL DNS name

nslookup web-app-clusterip
# nslookup -> asks the DNS server directly: "what IP does this name point to?"

exit
# Leave the test Pod (it gets auto-deleted because of --rm)
```

**What I saw:**
- Both the short name and the full DNS name returned the same Nginx welcome page.
- `nslookup web-app-clusterip` returned:
  ```
  Name:    web-app-clusterip.default.svc.cluster.local
  Address: 10.96.239.2
  ```
- This `10.96.239.2` is **exactly the same** as the `CLUSTER-IP` shown earlier by `kubectl get services` — confirming that DNS names and ClusterIPs point to the same place.

(You may also see some `** server can't find ...: NXDOMAIN` lines after the correct answer — this is normal. The DNS resolver inside the Pod tries a few extra name variations automatically; it already found the correct answer before trying those extra ones.)

**Screenshots:**

![DNS Short Name Test](2026/day-53/Screenshots/day53-6-dns-shortname-test.png)
![DNS Full Name Test](2026/day-53/Screenshots/day53-7-dns-fullname-test.png)
![nslookup DNS Match](2026/day-53/Screenshots/day53-8-nslookup-dns-match.png)

---

## Task 4: NodePort Service (External Access via Node IP)

A `NodePort` Service opens a specific port on **every Node** in the cluster, so that traffic can reach the Pods from outside the cluster (for example, from the same machine or network as the node).

**File: `nodeport-service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort                    # Opens a port on every Node
  selector:
    app: web-app                     # Same selector, targets the same 3 Pods
  ports:
  - port: 80                          # Port used inside the cluster (Service's own port)
    targetPort: 80                     # Port on the Pod to forward traffic to
    nodePort: 30080                    # The port opened on the NODE itself
                                         # (valid range is usually 30000-32767)
```

**Commands used:**

```bash
kubectl apply -f nodeport-service.yaml
# Creates the NodePort Service

kubectl get services
# Confirms the Service was created; PORT(S) column shows "80:30080/TCP"
# meaning: Service port 80 is mapped to Node port 30080

kubectl get nodes -o wide
# Lists all Nodes along with their INTERNAL-IP (needed to test the NodePort)
```

I found my Node's internal IP (`172.18.0.2`) from the `kubectl get nodes -o wide` output, and tested it directly:

```bash
curl http://172.18.0.2:30080
# curl -> sends an HTTP request directly to the Node's IP, on the NodePort we opened
```

**What I saw:** The Nginx welcome page loaded successfully — proving that traffic sent to `<Node-IP>:30080` correctly reaches one of the 3 Pods, from outside the Pod network.

**Screenshots:**

![NodePort Service Created](2026/day-53/Screenshots/day53-9-nodeport-svc-created.png)
![NodePort curl Success](2026/day-53/Screenshots/day53-10-nodeport-curl-success.png)

---

## Task 5: LoadBalancer Service (Cloud External Access)

In a real cloud environment (AWS, GCP, Azure), a `LoadBalancer` Service asks the cloud provider to create a **real external load balancer** with a public IP address, which routes traffic straight to the cluster.

**File: `loadbalancer-service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer                # Asks the cloud provider for a real external IP
  selector:
    app: web-app                     # Same selector, same 3 Pods
  ports:
  - port: 80                          # Port used by people connecting from outside
    targetPort: 80                     # Port on the Pod to forward to
```

**Commands used:**

```bash
kubectl apply -f loadbalancer-service.yaml
# Creates the LoadBalancer Service

kubectl get services
# Shows all Services, including the LoadBalancer's EXTERNAL-IP column
```

**What I saw:** The `EXTERNAL-IP` column showed `<pending>` and stayed that way.

**Why does this happen?**

My cluster is a **kind** (Kubernetes in Docker) cluster running on an AWS EC2 instance — it is a **local/self-managed cluster**, not a real managed cloud cluster. A `LoadBalancer` type needs an actual cloud provider plugin to create a real external IP address. Since there is no such cloud provider connected to this cluster, Kubernetes keeps waiting forever, and the status stays `<pending>`. This is **expected behavior**, not an error.

(On a real cloud cluster such as EKS, GKE, or AKS, this same YAML file would automatically get a real public IP or hostname from the cloud provider within a minute or two.)

**Screenshot:**

![LoadBalancer YAML and get services](2026/day-53/Screenshots/day53-12-loadbalancer-yaml-and-get-services.png)

---

## Task 6: Understand the Service Types Side by Side

| Type | Accessible From | Use Case |
|---|---|---|
| **ClusterIP** | Inside the cluster only | Internal communication between services |
| **NodePort** | Outside via `<NodeIP>:<NodePort>` | Development, testing, direct node access |
| **LoadBalancer** | Outside via cloud load balancer | Production traffic in cloud environments |

An important fact: these types **build on top of each other**.

```
LoadBalancer  -->  creates a NodePort  -->  which creates a ClusterIP
```

So a `LoadBalancer` Service actually contains **all three** underneath it.

**Commands used:**

```bash
kubectl get services -o wide
# Same as "get services" but also shows the SELECTOR column
# -> confirms all 3 Services use the same selector "app=web-app"
# -> meaning they all target the exact same 3 Pods

kubectl describe service web-app-loadbalancer
# Shows the FULL details of one specific Service
```

**What I saw in the `describe` output:**

```
IP:          10.96.160.37        <- this is the ClusterIP
NodePort:    <unset>  30853/TCP  <- this is the auto-created NodePort
Endpoints:   10.244.0.5:80, 10.244.0.7:80, 10.244.0.6:80   <- the actual 3 Pod IPs
```

This confirms that even though I only asked for a `LoadBalancer`, Kubernetes automatically gave it a `ClusterIP` and a `NodePort` as well — proving that `LoadBalancer` is built on top of `NodePort`, which is built on top of `ClusterIP`.

**Screenshot:**

![Describe LoadBalancer Layers](2026/day-53/Screenshots/day53-13-describe-loadbalancer-layers.png)

---

## Task 7: Clean Up

Once all tasks were verified, I deleted everything I created, so the cluster is back to a clean state.

**Commands used:**

```bash
kubectl delete -f app-deployment.yaml
# Deletes the Deployment (and all 3 Pods along with it)

kubectl delete -f clusterip-service.yaml
kubectl delete -f nodeport-service.yaml
kubectl delete -f loadbalancer-service.yaml
# Deletes all 3 Services, one by one

kubectl get pods
# Should now show "No resources found" — confirms all Pods are gone

kubectl get services
# Should now show ONLY the built-in "kubernetes" Service
# -> confirms all the Services I created are gone
```

**What I saw:**
- `kubectl get pods` → `No resources found in default namespace.`
- `kubectl get services` → only the default `kubernetes` Service remained.

This confirms the cleanup was successful.

**Screenshot:**

![Cleanup Verified](2026/day-53/Screenshots/day53-14-cleanup-verified.png)

---

## Summary — What I Learned Today

- Pods get random, unstable IPs — **Services** solve this by giving a stable address.
- **ClusterIP** (default): internal-only address, used for communication between apps inside the cluster.
- **NodePort**: opens a fixed port on every Node, used for quick external/dev access.
- **LoadBalancer**: asks the cloud for a real public IP, used for production traffic — on a local cluster like `kind` or Minikube (without a tunnel), it will stay `<pending>` since there's no real cloud provider.
- Every Service automatically gets a **DNS name** (`<service-name>.<namespace>.svc.cluster.local`), so Pods can talk to each other by name instead of by IP.
- `LoadBalancer` → is built on top of → `NodePort` → which is built on top of → `ClusterIP`. A single LoadBalancer Service actually carries all three underneath it.
