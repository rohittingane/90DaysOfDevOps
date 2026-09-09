# Day 49 — DevSecOps: Trivy, Secret Scanning, Dependency Review & Secure CI/CD
## Complete Merged Guide — Every Task, Every Line of Code, Every Error, Explained Simply

This is the complete, final Day 49 documentation. It combines:
- **What** was done (5 security tasks, with diagrams)
- **Why** each piece was needed
- **Every single line** of every workflow file and the Dockerfile, explained in simple English
- **Every error** that occurred, its exact root cause, and exactly how it was fixed
- **Screenshot references** placed right after the step they prove

Anyone with zero prior knowledge of this task can read this file top to bottom and complete the entire Day 49 task on their own.

---

# PART 1 — OVERVIEW

## 1. What was Day 49 about?

Before Day 49, the pipeline could already:
- Build the application
- Run tests
- Build Docker images
- Push Docker images to Docker Hub
- Deploy the application

On Day 49, **security** was added into this CI/CD process. The five tasks were:

1. Scan Docker images for vulnerabilities using **Trivy**
2. Enable **GitHub Secret Scanning**
3. Understand **Push Protection**
4. Add **Dependency Review** to the Pull Request pipeline (+ enable **Dependency Graph**)
5. Add **minimum required GitHub Actions permissions** (least privilege)

The core idea of DevSecOps:

> **Find security problems early, before they reach production.**

## 2. What is DevSecOps?

```text
Development + Security + Operations
```

In a traditional workflow, security is often checked only near the end (or after release). In **DevSecOps**, security checks are built directly into the normal development and CI/CD process — security becomes a regular step, not an afterthought.

The pipeline flow for this project became:

```text
Code
  ↓
Build
  ↓
Test
  ↓
Security Checks
  ↓
Docker Build
  ↓
Trivy Scan
  ↓
Docker Push
  ↓
Deploy
```

## 3. Repository structure (where every file lives)

```
github-actions-capstone/
├── .github/
│   └── workflows/
│       ├── main-pipeline.yml              → runs on push to main
│       ├── pr-pipeline.yml                → runs on pull requests
│       ├── reusable-build-test.yml        → reusable: install + test
│       ├── reusable-docker.yml            → reusable: build, scan, push Docker image
│       └── health-check.yml               → additional health-check workflow
├── Dockerfile                             → defines how the Docker image is built
├── requirements.txt                       → Python dependencies (e.g. Flask)
├── test-health.sh                         → test script that checks app health
├── README.md
└── app.py                                 → Flask app with a /health endpoint
```

**Path configuration note:** GitHub Actions ONLY looks for workflow files inside `.github/workflows/`. Any `.yml` file placed there is automatically detected. The file name itself doesn't matter to GitHub — what matters is the `on:` trigger written inside the file.

## 4. Final Day 49 architecture (complete flow)

```text
                         Pull Request
                              │
                              ▼
                       Build and Test
                              │
                              ▼
                     Dependency Review
                              │
                              ▼
                        PR Checks
                              │
                              ▼
                           Merge
                              │
                              ▼
                         main branch
                              │
                              ▼
                       Build and Test
                              │
                              ▼
                        Docker Build
                              │
                              ▼
                         Trivy Scan
                              │
                     ┌────────┴────────┐
                     │                 │
                    PASS              FAIL
                     │                 │
                     ▼                 ▼
                Docker Push       Stop Pipeline
                     │
                     ▼
                   Deploy


        Repository Security
              │
        ┌─────┴─────┐
        ▼           ▼
 Secret Scanning   Push Protection
```

---

# PART 2 — TASK 1: SCAN DOCKER IMAGES WITH TRIVY

## 5. What is Trivy?

Trivy is an **open-source security scanner**. It can scan:
- Docker images
- Operating-system packages
- Application dependencies
- Filesystems
- Git repositories
- Kubernetes-related configurations

Here, Trivy is used to scan the Docker image **before** it is pushed to Docker Hub.

## 6. Why does the image need scanning?

A Docker image is not only "our application." For example:

```dockerfile
FROM python:3.12-slim
```

This base image also brings along Python, Linux OS packages, and system libraries — any of which can contain **known vulnerabilities**. So the safe order is:

```text
Docker Build → Trivy Scan → Docker Push      ✅ (safe — check before releasing)
```

instead of:

```text
Docker Build → Docker Push → Trivy Scan      ❌ (unsafe — vulnerable image already public)
```

## 7. File: `reusable-docker.yml` — the file where all of this lives

**Path:** `.github/workflows/reusable-docker.yml`
**Purpose:** A **reusable workflow** (`workflow_call`) that builds the Docker image, scans it with Trivy, and only pushes it to Docker Hub if the scan passes. `main-pipeline.yml` calls this file twice — once for the `latest` tag, once for the commit-SHA tag.

### 7.1 Inputs and secrets

```yaml
on:
    workflow_call:
        inputs:
            image_name:
                required: true
                type: string
            tag:
                required: true
                type: string

        secrets:
            docker_username:
                required: true
            docker_token:
                required: true
```
- `workflow_call` means this file **cannot run by itself** — it only runs when another workflow calls it using `uses: ./path/to/this-file.yml`. This is what makes it "reusable."
- `inputs:` — `image_name` and `tag` are both `required: true`, meaning **whoever calls this file MUST supply them** — there's no default. If missing, the workflow fails immediately.
- `secrets:` — declares that this workflow needs `docker_username` and `docker_token` passed in from the caller. Secrets never auto-inherit into reusable workflows; they must be explicitly passed.

### 7.2 Permissions (Task 4 principle applied here too)

```yaml
permissions:
    contents: read
```
- Restricts this workflow to **read-only** access on the repository — it can check out code, but cannot write/modify repo contents. This is the "least privilege" principle: since this workflow only builds/scans/pushes a Docker image, it never needs write access.

### 7.3 The job and its steps

```yaml
jobs:
    docker-build-push:
        runs-on: ubuntu-latest

        outputs:
            image_url: ${{ steps.set_output.outputs.image_url }}
```
- `docker-build-push:` is the job name.
- `outputs: image_url` — exposes the final image path so any workflow that calls this one can read it later (used by the `deploy` job).

**Step 1 — Checkout code**
```yaml
        steps:
            - name: Checkout code
              uses: actions/checkout@v4
```
- Downloads (clones) the repository code onto the runner. Needed because the `Dockerfile` and app code must be present to build the image.

**Step 2 — Log in to Docker Hub**
```yaml
            - name: Log in to Docker Hub
              uses: docker/login-action@v3
              with:
                  username: ${{ secrets.docker_username }}
                  password: ${{ secrets.docker_token }}
```
- Authenticates with Docker Hub using the passed-in secrets. Required before any push is allowed.

**Step 3 — Build the image (but do NOT push yet)**
```yaml
            - name: Build Docker image
              run: |
                  docker build -t ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }} .
```
- Builds the image using the repo's `Dockerfile`. `-t` tags it as `username/image_name:tag`, e.g. `rohittingane/myapp:latest`.
- **Important:** the image is built but NOT pushed here. It's built first so there's an actual image to scan next. Push is deliberately delayed until AFTER the scan passes — this is the safe order from section 6.

**Step 4 — THE CORE DAY 49 STEP: Trivy scan**
```yaml
            - name: Scan Docker Image for Vulnerabilities
              uses: aquasecurity/trivy-action@master
              with:
                  image-ref: "${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}"
                  format: "table"
                  exit-code: "1"
                  severity: "CRITICAL,HIGH"
                  ignore-unfixed: true
```

Each setting explained one at a time:

- **`uses: aquasecurity/trivy-action@master`** — tells GitHub Actions to run the official Trivy Action.
- **`image-ref:`** — tells Trivy exactly which image to scan (the one just built), e.g. `rohittingane/myapp:latest`.
- **`format: "table"`** — prints results as a readable table in the log instead of raw JSON, e.g.:
  ```text
  Library       Vulnerability       Severity
  openssl       CVE-XXXX            HIGH
  package-x     CVE-YYYY            CRITICAL
  ```
- **`exit-code: "1"`** — this is the most important setting. If Trivy finds matching vulnerabilities, it exits with code `1`, which GitHub Actions treats as a **failure**. This makes Trivy act as a **security gate**:
  ```text
  Vulnerability found → Trivy exit code 1 → GitHub Actions FAIL → Docker Push does NOT run
  ```
- **`severity: "CRITICAL,HIGH"`** — only CRITICAL and HIGH severity issues count toward failing the step. LOW/MEDIUM issues are still reported in the log but don't block the pipeline.
- **`ignore-unfixed: true`** — **the exact Day 49 task requirement.** Some vulnerabilities are known but have **no available fix yet** (the maintainers haven't released a patch). Without this setting, the pipeline could fail forever on issues nobody can currently do anything about. With `ignore-unfixed: true`, Trivy skips those from the failure count — but still fails on anything CRITICAL/HIGH that **does** have an available fix.
  > **Important nuance:** `ignore-unfixed: true` does NOT mean the vulnerability doesn't exist. It only means unfixed vulnerabilities are not used to fail this particular security gate.

**Step 5 — Push the image (only runs if Trivy passed)**
```yaml
            - name: Push Docker image
              run: |
                  docker push ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}
```
- Pushes the image to Docker Hub. Notice there's **no `if:` condition** — GitHub Actions automatically skips all remaining steps in a job once an earlier step fails, so this naturally never runs if Trivy failed.

**Step 6 — Save the output**
```yaml
            - name: Set output
              id: set_output
              run: |
                  echo "image_url=${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}" >> $GITHUB_OUTPUT
```
- Saves the full image path into `$GITHUB_OUTPUT` under the key `image_url`. This exact value is later read by the `deploy` job in `main-pipeline.yml`.

### 7.4 Final safe order this creates

```text
Docker Build ✅ → Trivy Scan ✅ → Docker Push ✅ → Deploy
```

If Trivy fails:
```text
Docker Build ✅ → Trivy Scan ❌ → Docker Push ⏭️ Skipped
```

## 8. First Trivy run — what actually happened

After adding the Trivy step, the pipeline was triggered. The **first scan reported**:

```text
Total: 54
HIGH: 51
CRITICAL: 3
```

**The pipeline failed.** This was NOT a YAML syntax error — the security gate was working exactly as configured. Because `exit-code: "1"` was set, Trivy correctly returned a failure since HIGH/CRITICAL vulnerabilities were found.

**📸 Screenshot:** `task1-trivy-scan-output.png` — shows the raw Trivy scan output with the `Total: 54 / HIGH: 51 / CRITICAL: 3` result.

**📸 Screenshot:** `task1-pipeline-failed-summary.png` — shows the pipeline marked as failed because of the Trivy security check.

**📸 Screenshot:** `task1-pipeline-triggered.png` — shows the pipeline execution starting after the security changes were added.

### What happened to Docker Push?
Because Trivy failed, the push step never executed — exactly the intended behavior:
```text
Docker Build   ✅
Trivy Scan     ❌
Docker Push    ⏭️ Skipped
```

## 9. Understanding the error — what to do about 54 vulnerabilities

`54 vulnerabilities` does not automatically mean the Dockerfile is broken — it means Trivy found known vulnerabilities in components inside the image. The next step is to figure out whether they have available fixes. Possible actions:
- Update the base image
- Update OS packages
- Update application dependencies
- Use a newer package version
- Ignore only vulnerabilities that currently have no fix (`ignore-unfixed: true`)
- Check if the vulnerable package is even required

## 10. Fix Part 1 — Update the Dockerfile

**Path:** `Dockerfile` (repository root)

### Full code

```dockerfile
FROM python:3.12-slim
WORKDIR /app

# Day 49 (DevSecOps): Update all system packages to pull in the latest
# security patches for the base OS (fixes many CVEs found by Trivy,
# e.g. in openssl, util-linux, perl, etc.)
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

### Line-by-line explanation

**`FROM python:3.12-slim`**
- Sets the **base image** — the starting point everything else is built on. `python:3.12-slim` is an official Python 3.12 image built on a lightweight Debian base. "Slim" means fewer unnecessary packages are included — smaller image, smaller attack surface (fewer packages = fewer potential CVEs).

**`WORKDIR /app`**
- Sets the working directory inside the container to `/app`. Every following `COPY`, `RUN`, or `CMD` executes relative to this folder. Docker creates `/app` automatically if it doesn't already exist.

**`RUN apt-get update && apt-get upgrade -y && apt-get clean && rm -rf /var/lib/apt/lists/*`** — **the main Day 49 Dockerfile fix.**
Even an official base image's underlying Debian OS packages (like `openssl`, `util-linux`, `perl`) can have known CVEs if the image was built a while back and hasn't been refreshed. Trivy scans exactly these OS-level packages, not just the Python code. Breaking down each part (joined with `&&` so they all run in a single Docker layer, keeping the image smaller):
  - `apt-get update` — refreshes the local catalogue of available package versions from Debian's servers. Installs nothing yet — just updates the list.
  - `apt-get upgrade -y` — actually installs the newest available version of every already-installed package, pulling in security patches. `-y` auto-confirms any prompts (needed since Docker builds run non-interactively).
  - `apt-get clean` — deletes cached `.deb` package files that are no longer needed after installation.
  - `rm -rf /var/lib/apt/lists/*` — deletes the downloaded package-list files; keeping them around adds nothing useful at runtime and only bloats the image.
- **Why this matters for Trivy:** most of the 51 HIGH / 3 CRITICAL vulnerabilities were likely in outdated OS-level libraries, not the Python app code. Patching the OS at build time means Trivy finds far fewer real, fixable issues — and whatever remains unfixed is now correctly handled by `ignore-unfixed: true`.

**`COPY requirements.txt .` then `RUN pip install --no-cache-dir -r requirements.txt`**
- Copies just `requirements.txt` first, then installs the Python packages listed in it (`--no-cache-dir` skips storing pip's download cache, keeping the image smaller).
- **Why copy `requirements.txt` before the rest of the code?** Docker caches build layers. Since `requirements.txt` changes far less often than app code, keeping this step separate from `COPY . .` means Docker can skip re-running `pip install` on every code change — only rerunning it when `requirements.txt` itself changes. This makes rebuilds much faster.

**`COPY . .`**
- Copies everything else in the repository (the actual Flask app code, etc.) into `/app`.

**`EXPOSE 5000`**
- Documents that the app listens on port 5000. This alone does not publish the port — running the container still needs `-p 5000:5000` to actually expose it externally.

**`CMD ["python", "app.py"]`**
- The default command run when a container starts from this image — starts the Flask app. Unlike `RUN` (executes during build), `CMD` only executes when the container actually starts.

## 11. Fix Part 2 — Local Trivy verification (testing before pushing to CI)

Trivy was also tested **locally** to confirm the fix worked before relying on GitHub Actions:

```bash
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest \
  image \
  --severity CRITICAL,HIGH \
  --ignore-unfixed \
  trivy-test
```

Explanation of the important flags:
- `image` — tells Trivy to scan a Docker image (as opposed to a filesystem or repo).
- `--severity CRITICAL,HIGH` — same severity filter as in the CI pipeline.
- `--ignore-unfixed` — same "ignore unfixed" logic as in the CI pipeline, tested locally first.

**Local scan result:**
```text
Debian 13.6       0 vulnerabilities
Python metadata   0 vulnerabilities
```
The command completed successfully — meaning under these scan conditions (CRITICAL/HIGH + fixed-only), there were zero vulnerabilities. This does **not** mean every possible vulnerability of every severity is zero — only that nothing meeting the configured failure criteria remained.

## 12. Trivy scan after the fix — pipeline succeeds

After applying both the Dockerfile update and `ignore-unfixed: true`, the GitHub Actions pipeline was re-run.

**📸 Screenshot:** `task1-trivy-scan-after-fix.png` / `day49-trivy-fix-pipeline-success.png` — the full `main-pipeline.yml` run now shows all green: `test` → `docker-latest` + `docker-sha` (parallel) → `deploy`, total duration ~1m 1s.

**📸 Screenshot:** `day49-trivy-scan-log-details.png` / `task1-trivy-scan-log-details.png` — the detailed Trivy scan log, showing sub-steps like "Install Trivy," "Get current date," "Restore DB from cache," "Set GitHub Path," "Set Trivy environment variables," and "Run Trivy" (9s) — all succeeding. This log view is useful when investigating package name, vulnerability ID, severity, installed version, fixed version, and vulnerability status.

The safe flow now completes fully:
```text
Docker Build ✅ → Trivy Scan ✅ → Docker Push ✅
```

**📸 Screenshot:** `day49-dockerhub-myapp-tags.png` — confirms the image was actually pushed to Docker Hub (`rtingane2611/myapp`, tags `latest` and `sha-cf19f5f...`), proving the entire Build → Scan → Push chain worked end-to-end.

---

# PART 3 — TASK 2: GITHUB SECRET SCANNING & PUSH PROTECTION

## 13. What is a "secret"?

A secret is sensitive information such as:
```text
AWS Access Key
AWS Secret Access Key
GitHub Token
API Token
Database Password
Cloud Credentials
```
Example of what should never be committed:
```text
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
```

## 14. Why are leaked secrets dangerous?

If a credential (like an AWS key) is accidentally committed and exposed, anyone who finds it could use it. Possible consequences:
- Unauthorized AWS access
- Data exposure
- Unexpected resource creation
- Unexpected cloud billing charges
- Infrastructure compromise

## 15. Enabling Secret Scanning

Path in GitHub UI:
```text
Repository → Settings → Code security and analysis → Secret Protection → Secret scanning
```
Enable it here.

## 16. Verifying Secret Scanning

Path:
```text
Security and quality → Secret scanning → Alerts
```
Result: **"No secrets found."** This means no unresolved detected secrets existed in the repository at the time of checking.

**📸 Screenshot:** `task2-secret-scanning.png` — shows the Secret Scanning configuration/result.

## 17. What is Push Protection?

Push Protection is designed to actively **block** supported secrets from ever being pushed to GitHub in the first place — it's a step beyond detection.

```text
Secret Scanning   → Detect (after the fact)
Push Protection   → Prevent (before it even lands)
```

Flow when Push Protection is active:
```text
Developer → git push → GitHub detects supported secret → Push Protection → Push blocked
```

## 18. What to do if a real secret gets leaked anyway

1. Remove it from the code.
2. **Revoke or rotate the credential** (this is the critical step).
3. Create a new credential if required.
4. Store it securely going forward.
5. Check whether the exposed credential was actually used by someone else.
6. Review cloud activity logs if necessary.

> **Important:** Removing the secret from the latest commit does NOT make an already-exposed credential safe. Git history still contains it. The credential itself must be rotated/revoked.

## 19. Using GitHub Actions Secrets correctly

For CI/CD credentials, always use GitHub Secrets:
```text
Settings → Secrets and variables → Actions
```
Example secret names used in this project:
```text
DOCKER_USERNAME
DOCKER_TOKEN
```
Accessed inside workflows as:
```yaml
${{ secrets.DOCKER_TOKEN }}
```
Never hard-code real credentials directly into YAML files.

---

# PART 4 — TASK 3: DEPENDENCY REVIEW

## 20. What is Dependency Review?

Applications use external libraries — for example, a Python app might use `Flask`, `requests`, `gunicorn`. These dependencies can contain known vulnerabilities. **Dependency Review** checks dependency changes introduced through a Pull Request, before it's merged:

```text
Pull Request → Dependency Change → Security Check → PASS / FAIL
```

## 21. Dependency Graph (a prerequisite)

Dependency Review needs GitHub's **Dependency Graph** feature to actually work — it's what lets GitHub compare "old dependencies" vs "new dependencies" in a PR.

Path:
```text
Settings → Code security and analysis → Dependency graph
```
Enable it here.

## 22. File: `pr-pipeline.yml` — full explanation

**Path:** `.github/workflows/pr-pipeline.yml`
**Purpose:** Runs automatically when a Pull Request is opened or updated (before merge). Acts as a gatekeeper checking code quality and security before anything reaches `main`.

### 22.1 Trigger

```yaml
on:
    pull_request:
        branches:
            - main
        types:
            - opened
            - synchronize
```
- `pull_request:` — triggers on PR activity (different from `push:`).
- `branches: [main]` — only runs for PRs targeting `main`.
- `types:` — specifies which events trigger it:
  - `opened` — when a new PR is first created.
  - `synchronize` — when new commits are pushed to an already-open PR (so checks automatically re-run on every update).

### 22.2 Job 1 — reuse the test workflow

```yaml
jobs:
    call-build-test:
        uses: ./.github/workflows/reusable-build-test.yml
        with:
            python_version: "3.12"
            run_tests: true
```
- Calls the exact same reusable testing workflow used in `main-pipeline.yml`, so PRs get tested identically to production pushes.

### 22.3 Job 2 — the Dependency Review job

```yaml
    dependency-review:
        name: Dependency Review
        runs-on: ubuntu-latest
        permissions:
            contents: read
        steps:
            - name: Check Dependencies for Vulnerabilities
              uses: actions/dependency-review-action@v4
              with:
                  fail-on-severity: critical
```
Configuration explained piece by piece:
- **`dependency-review:`** — the internal job ID.
- **`name: Dependency Review`** — the display name shown in the GitHub UI.
- **`runs-on: ubuntu-latest`** — GitHub provides an Ubuntu runner.
- **`permissions: contents: read`** — this job only needs to read repository info (least privilege — Task 4's principle applied here too).
- **`uses: actions/dependency-review-action@v4`** — runs GitHub's official Dependency Review Action.
- **`fail-on-severity: critical`** — the security gate only fails if a **CRITICAL**-level vulnerable dependency is introduced by the PR.
- This job runs **in parallel** with `call-build-test` (it doesn't `need` anything).

### 22.4 Job 3 — the PR comment job

```yaml
    pr-comment:
        needs: call-build-test
        runs-on: ubuntu-latest
        permissions:
            contents: read
        steps:
            - name: Print summary
              run: 'echo "PR checks passed for branch: ${{ github.head_ref }}"'
```
- `needs: call-build-test` — waits for tests to pass first.
- `github.head_ref` — built-in variable holding the PR's source branch name (e.g. `test-pr-pipeline`).
- Just prints a confirmation summary line in the log.

## 23. First Dependency Review failure

A test PR was created: `Day 49: Test dependency review` (branch `test-pr-pipeline` → `main`).

The **first run** of Dependency Review failed with:
```text
Error: Dependency review is not supported on this repository.
Please ensure that Dependency graph is enabled.
```

**Root cause:** **Dependency Graph** was disabled at the repository level — Task 3's own prerequisite (section 21) hadn't been satisfied yet. This is a **repository configuration issue**, not a YAML syntax error.

**📸 Screenshot:** `day49-pr-dependency-review-failing.png` / `task2-secret-scanning.png` — shows **"Some checks were not successful — 1 failing, 2 successful checks"**, with `PR Pipeline / Dependency Review (pull_request)` marked **"Failing after 4s."**

**📸 Screenshot:** `day49-advanced-security-settings.png` — the Advanced Security settings page: at the time of the failure, **Dependency graph** and **Dependabot alerts** were not both properly enabled/populated.

## 24. Fixing the Dependency Review error

1. Went to **Settings → Security → Advanced Security**.
2. Enabled **Dependency graph**.
3. Enabled **Dependabot alerts**, so GitHub could actively track and report vulnerable dependencies.
4. This triggered GitHub to automatically run a **Dependabot dependency graph update** in the background.

**📸 Screenshot:** `day49-dependabot-pip-graph-update.png` — shows an automatic run named **"Graph Update: pip in /. #1"**, triggered by `dependabot[bot]`, succeeded in 46s. Result:

| Directory | Status | Details |
|---|---|---|
| `/` | ✅ Ok | Found 1 dependencies |

This confirms GitHub successfully built a working dependency graph for the Python (`pip`) ecosystem in the repo root.

5. The PR's checks were **re-run**.

**📸 Screenshot:** `day49-pr-pipeline-dependency-review-fixed.png` — re-run (`Latest #2`) of the PR Pipeline: `call-build-test / build-and-test` (11s) → `pr-comment` (3s), and **`Dependency Review` now succeeds in 6s** — fully fixed.

6. Final result:
```text
Dependency Review
No vulnerabilities or license issues or OpenSSF Scorecard issues found.
```

### Why did it say "Scanned Files: None"?
This particular test PR didn't actually introduce a dependency-file change, so the scan result naturally showed:
```text
Scanned Files
None
```
This is expected for a PR that doesn't touch `requirements.txt` — the important proof point was that the **Dependency Review Action itself successfully ran** once Dependency Graph was enabled. For a real dependency-vulnerability test, `requirements.txt` itself would need to change in the PR.

**📸 Screenshot:** `day49-compare-pr-dependency-review.png` — the compare/diff view before creating the PR, showing 1 file changed (`README.md`).

**📸 Screenshot:** `day49-pr-branch-pushed.png` — shows GitHub detecting the `test-pr-pipeline` branch after a push, prompting "Compare & pull request."

## 25. PR becomes mergeable and gets merged

**📸 Screenshot:** `day49-pr-ready-to-merge.png` — the PR marked **"Ready to merge"** with **"All checks have passed — 3 successful checks"** and no conflicts with the base branch.

**📸 Screenshot:** `day49-pr-merged-successfully.png` — the PR status changed to **"Merged"**, commit `a891bf2` merged into `main`, note: **"5 of 6 checks passed"** (the merge itself triggered `main-pipeline.yml` to run again automatically — an additional set of checks beyond the 3 PR-level checks, hence 6 total across both pipelines).

---

# PART 5 — TASK 4: GITHUB ACTIONS PERMISSIONS (LEAST PRIVILEGE)

## 26. What are GitHub Actions permissions?

GitHub Actions uses a special token called `GITHUB_TOKEN` to interact with GitHub resources on the workflow's behalf. A workflow can be granted permissions such as:
```text
contents
pull-requests
issues
actions
```
Giving a workflow unnecessary **write** permissions increases security risk — if that workflow (or an action it calls) is ever compromised or misused, broader permissions mean broader potential damage.

## 27. What is "least privilege"?

> **Give a workflow only the permissions it actually needs — nothing more.**

For example:
```yaml
permissions:
    contents: read
```
is better than leaving the default (broader) permissions if the workflow only needs to inspect/check out files.

## 28. Where this was applied in this project

**In `reusable-docker.yml`:**
```yaml
permissions:
    contents: read
```
This workflow only needs to check out and read the repository (to access the `Dockerfile` and app code) — it never modifies repository files. So `contents: read` is exactly right.

**In `pr-pipeline.yml`'s `dependency-review` job:**
```yaml
permissions:
    contents: read
```
The Dependency Review process only needs to read repository information (the dependency files) — it doesn't need to modify anything.

## 29. When would broader permissions actually be needed?

If a workflow needed to **write a comment** onto a Pull Request (not just print a log line), it would need something like:
```yaml
permissions:
    contents: read
    pull-requests: write
```
Example flow: `Workflow → Analyse PR → Post comment on PR`. But since the `pr-comment` job in this project only **prints** a summary in the log (it doesn't actually post anything back to the PR), `pull-requests: write` is not needed — keeping permissions as small as possible.

---

# PART 6 — TASK 5: THE COMPLETE SECURE PIPELINE

## 30. Full Pull Request flow

```text
Pull Request
      ↓
Build and Test
      ↓
Dependency Review
      ↓
PR checks
      ↓
PASS
```
Only after all checks pass should the PR be merged.

## 31. Full main-branch flow (`main-pipeline.yml`)

**Path:** `.github/workflows/main-pipeline.yml`
**Purpose:** Runs whenever code is pushed to `main` (including when a PR is merged — a merge counts as a push).

```text
Merge to main
      ↓
Build and Test
      ↓
Docker Build (×2, parallel — "latest" tag and "sha-<commit>" tag)
      ↓
Trivy Scan
      ↓
Docker Push
      ↓
Deploy
```

### Full code, explained line by line

```yaml
name: Main Pipeline
on:
    push:
        branches:
            - main
```
- Names the workflow (shown in the Actions tab). Triggers only on pushes to `main` — other branches (like `test-pr-pipeline`) never trigger this file.

```yaml
jobs:
    test:
        uses: ./.github/workflows/reusable-build-test.yml
        with:
            python_version: "3.12"
            run_tests: true
```
- The `test` job reuses `reusable-build-test.yml`, passing Python 3.12 and telling it to actually run tests.

```yaml
    docker-latest:
        needs: test
        uses: ./.github/workflows/reusable-docker.yml
        with:
            image_name: myapp
            tag: latest
        secrets:
            docker_username: ${{ secrets.DOCKER_USERNAME }}
            docker_token: ${{ secrets.DOCKER_TOKEN }}
```
- `needs: test` — waits for tests to pass first; a broken build never gets shipped.
- Calls `reusable-docker.yml`, tagging the image `latest`. Passes Docker Hub credentials from repository secrets (never hard-coded).

```yaml
    docker-sha:
        needs: test
        uses: ./.github/workflows/reusable-docker.yml
        with:
            image_name: myapp
            tag: sha-${{ github.sha }}
        secrets:
            docker_username: ${{ secrets.DOCKER_USERNAME }}
            docker_token: ${{ secrets.DOCKER_TOKEN }}
```
- Same as `docker-latest`, but tagged with `sha-${{ github.sha }}` — the exact commit ID. Runs **in parallel** with `docker-latest` (both only depend on `test`).
- Why tag with SHA too? `latest` gets overwritten on every push — you lose track of exactly which version is running. The SHA-tagged image lets you **roll back** to an exact previous version if something breaks.

```yaml
    deploy:
        needs: [docker-latest, docker-sha]
        runs-on: ubuntu-latest
        environment:
            name: production
        steps:
            - name: Deploy application
              run: 'echo "Deploying image: ${{ needs.docker-latest.outputs.image_url }} to production"'
```
- Waits for **both** Docker jobs to succeed.
- `environment: name: production` links this job to a GitHub Environment, which can (in Settings) be configured to require manual approval — protecting production from accidental deploys.
- The step is a **simulated deploy** — it just echoes a message, using `image_url` output that was set inside `reusable-docker.yml` and passed through the `docker-latest` job.

**📸 Screenshot:** `day49-deploy-job-success.png` — shows the `deploy` job's log: `Deploying image: <username>/myapp:latest to production`.

## 32. File: `reusable-build-test.yml` — full explanation

**Path:** `.github/workflows/reusable-build-test.yml`
**Purpose:** Shared reusable workflow — installs Python, installs dependencies, runs the health-check test. Both `main-pipeline.yml` and `pr-pipeline.yml` call this same file, so testing logic is written once and reused everywhere.

```yaml
on:
  workflow_call:
    inputs:
      python_version:
        required: false
        type: string
        default: "3.12"

      run_tests:
        required: false
        type: boolean
        default: true
```
- `python_version` — text input, optional, defaults to `"3.12"` if the caller doesn't specify one.
- `run_tests` — boolean input, optional, defaults to `true`.

```yaml
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    outputs:
      test_result: ${{ steps.run_tests_step.outputs.result }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
```
- Job name `build-and-test`, exposes `test_result` as an output for the caller. Step 1 downloads the repo code.

```yaml
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python_version }}
```
- Installs Python — whichever version the caller passed in.

```yaml
      - name: Install dependencies
        run: pip install -r requirements.txt
```
- Installs every package listed in `requirements.txt` (e.g. Flask).

```yaml
      - name: Run tests
        id: run_tests_step
        if: ${{ inputs.run_tests == true }}
        run: |
          chmod +x test-health.sh
          if ./test-health.sh; then
            echo "result=passed" >> $GITHUB_OUTPUT
          else
            echo "result=failed" >> $GITHUB_OUTPUT
          fi
```
- `id: run_tests_step` names the step so its output is readable later.
- `if:` — this step only runs when `run_tests` is `true`; can be entirely skipped by a caller if needed.
- `chmod +x test-health.sh` — gives the test script execute permission (Linux requires this explicitly).
- The `if/else` runs the script and writes `result=passed` or `result=failed` into `$GITHUB_OUTPUT`, making it readable elsewhere as `steps.run_tests_step.outputs.result`.

## 33. The complete security gate (visual)

```text
Docker Build
      ↓
   Trivy
   ┌──┴──┐
   ↓     ↓
 PASS   FAIL
   ↓     ↓
 Push   STOP
```
This gate guarantees the normal pipeline path can never push a vulnerable (CRITICAL/HIGH, fixable) image to Docker Hub.

## 34. Final proof screenshots

**📸 Screenshot:** `day49-deploy-job-success.png` — successful deploy job after all security checks passed.

**📸 Screenshot:** `day49-dockerhub-myapp-tags.png` — final Docker image available on Docker Hub after the scan passed, confirming the order Build → Security Scan → PASS → Push.

**📸 Screenshot:** `day49-advanced-security-settings.png` — the repository's security settings page (Dependency Graph, Secret Scanning, Push Protection, and related security features) as configured for Day 49.

## 35. Complete Day 49 workflow (final combined diagram)

```text
Developer writes code
        │
        ▼
    Pull Request
        │
        ▼
   Build and Test
        │
        ▼
 Dependency Review
        │
        ▼
    PR Checks
        │
        ▼
      Merge
        │
        ▼
      main
        │
        ▼
 Build and Test
        │
        ▼
  Docker Build
        │
        ▼
   Trivy Scan
        │
   ┌────┴────┐
   │         │
 PASS       FAIL
   │         │
   ▼         ▼
Push       Stop
   │
   ▼
Deploy
```

At the repository level, running independently of every pipeline:
```text
Secret Scanning + Push Protection
```

---

# PART 7 — ERRORS FACED (SUMMARY) AND WARNINGS SEEN

## 36. Error 1 — Trivy found 54 vulnerabilities

**Result:**
```text
Total: 54
HIGH: 51
CRITICAL: 3
```

**Why:** The Docker image (base OS + packages) contained known CVEs.

**What happened:** Because `exit-code: "1"` was configured, Trivy correctly failed the workflow — this was the security gate working as designed, not a bug.

**Result of the failure:** Docker Push was automatically skipped.

**Fix:**
1. Checked each vulnerability's fix-availability status.
2. Added `ignore-unfixed: true` to stop failing on vulnerabilities with no available patch.
3. Updated OS packages in the `Dockerfile` (`apt-get update && apt-get upgrade -y && apt-get clean && rm -rf /var/lib/apt/lists/*`) to actually patch fixable vulnerabilities before Trivy even scans.

## 37. Error 2 — Dependency Review not supported

**Error message:**
```text
Dependency review is not supported on this repository.
Please ensure that Dependency graph is enabled.
```

**Cause:** Dependency Graph was disabled at the repository level.

**Fix:** Enabled it via `Settings → Code security and analysis → Dependency graph`, then enabled **Dependabot alerts**, which triggered GitHub to build a working dependency graph. Re-ran Dependency Review.

**Result:**
```text
Dependency Review
        ↓
     Success
```

## 38. Warning (not an error) — Node.js 20 deprecated

The Dependency Review job displayed a warning that one of its actions was targeting Node.js 20, while GitHub runners were moving toward Node.js 24. This was **only a warning** — it was not the cause of the Dependency Review failure, and the job still completed successfully. It was correctly not treated as the root cause of the earlier failure.

## 39. Warning (not an error) — "Secret in output"

The reusable Docker workflow showed a warning similar to:
```text
Skip output 'image_url' since it may contain secret
```
This happened because the `image_url` output value was constructed using a value derived from GitHub Secrets (the Docker username). This is only a warning — the workflow still completed successfully and the deploy job could still read the value correctly.

---

# PART 8 — SECURITY LESSONS AND SUMMARY

## 40. Important security lessons

**Never commit real secrets.** Never write things like:
```yaml
AWS_SECRET_ACCESS_KEY: "real-secret"
```
or
```text
password=real-password
```
Always use GitHub Secrets or a proper secret manager instead.

**Do not blindly ignore vulnerabilities just to make the pipeline green.** Before adding an ignore rule, always understand:
```text
What package?
What CVE?
What severity?
Is there a fix?
Can the package be updated?
Is the package even required?
```

**Keep base images updated.** Even official, well-maintained base images accumulate new CVEs over time. Security scanning should be repeated regularly, since new vulnerabilities can be discovered after an image was originally built — a scan that passed last month might not pass today.

## 41. What actually changed for Day 49 (final summary table)

| File | Change | Why |
|---|---|---|
| `reusable-docker.yml` | Added the entire **Trivy scan step** | To actually scan the image for vulnerabilities before pushing |
| `reusable-docker.yml` | Added `ignore-unfixed: true` | So the pipeline doesn't fail forever on vulnerabilities with no available patch — the exact Day 49 task requirement |
| `reusable-docker.yml` | Added `exit-code: "1"` and `severity: "CRITICAL,HIGH"` | Only serious, fixable issues actually block the pipeline |
| `reusable-docker.yml` | Added `permissions: contents: read` at workflow level | Least-privilege security |
| `pr-pipeline.yml` | Added `permissions: contents: read` on jobs | Same least-privilege principle applied to PR checks |
| `Dockerfile` | Added `apt-get update && apt-get upgrade -y && apt-get clean && rm -rf /var/lib/apt/lists/*` | Patches OS-level CVEs (openssl, util-linux, perl, etc.) at build time, before Trivy even scans |
| Repo Settings | Enabled **Secret Scanning** | Detects accidentally committed credentials |
| Repo Settings | Reviewed **Push Protection** | Actively blocks supported secrets from ever being pushed |
| Repo Settings | Enabled **Dependency Graph** | Prerequisite for Dependency Review to work at all |
| Repo Settings | Enabled **Dependabot alerts** | Fixed the Dependency Review check by giving it a working dependency graph to compare against |

## 42. What was learned on Day 49

- **Trivy:** How to scan Docker images for known vulnerabilities and use the scan as a CI/CD security gate.
- **Secret Scanning:** Why credentials must never be committed to Git, and how GitHub detects supported secret types.
- **Push Protection:** How it can proactively prevent supported secrets from ever entering the repository.
- **Dependency Review:** How to check dependency changes introduced by a Pull Request, before merge.
- **Dependency Graph:** That Dependency Review requires this graph to exist and be populated — a prerequisite, not optional.
- **GitHub Actions Permissions:** Workflows should always use the minimum permissions they actually need (least privilege).
- **DevSecOps (overall):** Security should be a normal part of development and deployment — not something checked only after the fact.

## 43. Before vs. After Day 49

**Before:**
```text
Code → Build → Test → Docker Build → Docker Push → Deploy
```

**After:**
```text
Pull Request → Build + Test → Dependency Review → PR Checks → Merge
      → Build + Test → Docker Build → Trivy Scan → Docker Push → Deploy
```

Plus, running independently at the repository level:
```text
Secret Scanning + Push Protection
```

> **The main lesson from Day 49: Build it, test it, secure it, then release it.**

## 44. Quick checklist to redo this task yourself

- [ ] Update the `Dockerfile` to run `apt-get update && apt-get upgrade -y && apt-get clean && rm -rf /var/lib/apt/lists/*` right after `FROM`, to patch OS-level CVEs before scanning.
- [ ] Add the Trivy scan step to `reusable-docker.yml`, with `ignore-unfixed: true`, `severity: "CRITICAL,HIGH"`, and `exit-code: "1"`, placed **between** the Docker build and Docker push steps.
- [ ] Add `permissions: contents: read` at the top of `reusable-docker.yml`.
- [ ] Enable **Secret Scanning** in repo Settings → Code security and analysis.
- [ ] Review **Push Protection** settings.
- [ ] Enable **Dependency Graph** in repo Settings — required before Dependency Review can work.
- [ ] Enable **Dependabot alerts** if Dependency Review fails with "Dependency review is not supported."
- [ ] Add a `dependency-review` job to `pr-pipeline.yml` using `actions/dependency-review-action@v4` with `fail-on-severity: critical` and `permissions: contents: read`.
- [ ] Open a test PR and confirm all checks pass (build-and-test, Dependency Review, pr-comment).
- [ ] Merge the PR and confirm `main-pipeline.yml` runs end-to-end (test → docker-latest + docker-sha → deploy).
- [ ] Check Docker Hub to confirm both `latest` and `sha-...` tags were pushed after the scan passed.

---

#
