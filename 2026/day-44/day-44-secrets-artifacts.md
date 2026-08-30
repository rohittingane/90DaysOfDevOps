# Day 44 - Secrets, Artifacts & Running Real Tests in CI

---

## Task 1: GitHub Secrets

Created a secret `MY_SECRET_MESSAGE` in repo Settings → Secrets and Variables → Actions.
Created a workflow that safely prints `The secret is set: true` without exposing the value.

Tried printing the secret directly using `${{ secrets.MY_SECRET_MESSAGE }}`.
GitHub automatically masked the value in the logs and printed `***` instead.

### Why should you never print secrets in CI logs?

- CI logs are often visible to multiple team members, and sometimes even publicly (in open-source repos).
- If a secret gets printed as plain text, anyone with log access can copy and misuse it (e.g. database passwords, API tokens, cloud credentials).
- Logs can get archived, exported, or cached in places you don't control - once a secret leaks into a log, you often have to rotate/invalidate it.
- GitHub masks known secret values automatically, but this masking only works if the secret is used through `secrets.*` - if you build a secret manually inside a variable, masking is not guaranteed.

**Secret created in GitHub Settings:**
![Secret created](Screenshots/day44-secret-created.png)

**Workflow code:**
![Workflow code](Screenshots/day44-workflow-code.png)

**Workflow run - success:**
![Workflow success](Screenshots/day44-workflow-success.png)

**Secret masking proof in logs:**
![Secret masking proof](Screenshots/day44-secret-masking-proof.png)

**Run summary:**
![Workflow summary](Screenshots/day44-workflow-summary.png)

---

## Task 2: Secrets as Environment Variables

Added `DOCKER_USERNAME` and `DOCKER_TOKEN` as repository secrets (to be used in Day 45 for Docker login).

Created a workflow step that passes `DOCKER_USERNAME` secret as an environment variable (`DOCKER_USER`)
and used it inside a shell command without hardcoding the value.

Verified that GitHub automatically masks the secret value in logs, even when used through an
environment variable - output showed `Logging in as user: ***` instead of the real username.

**DOCKER_USERNAME secret created:**
![Docker username secret](Screenshots/day44-docker-username-secret.png)

**DOCKER_TOKEN secret created:**
![Docker token secret](Screenshots/day44-docker-token-secret.png)

**Workflow code:**
![Env secrets code](Screenshots/day44-env-secrets-code.png)

**First run (before DOCKER_USERNAME had a value issue was fixed):**
![Env secrets run success](Screenshots/day44-env-secrets-success.png)

**Final run - secret properly masked in logs:**
![Env secrets final success](Screenshots/day44-env-secrets-final-success.png)

---

## Task 3: Upload Artifacts

Created a workflow step that generates a `report.txt` file with build info (date, status).
Used `actions/upload-artifact@v4` to save the file as an artifact named `build-report`.

Verified in the Actions run summary that the artifact appeared under the "Artifacts" section (198 Bytes).
Downloaded the artifact from the Actions tab and confirmed the file content matched exactly
what was generated during the workflow run.

**Workflow code:**
![Artifacts workflow code](Screenshots/day44-artifacts-code.png)

**Artifact uploaded (visible in Actions run):**
![Artifact uploaded](Screenshots/day44-artifact-uploaded.png)

**Artifact downloaded and verified:**
![Artifact downloaded](Screenshots/day44-artifact-downloaded.png)

---

## Task 4: Download Artifacts Between Jobs

Created a workflow with two jobs:
- `create-file`: generates a file (`data.txt`) and uploads it as an artifact named `shared-file`
- `read-file`: uses `needs: create-file` to run after the first job, downloads the same artifact,
  and prints its content using `cat`

Verified in the run summary that the artifact (`shared-file`, 149 Bytes) was passed between jobs
correctly, and the `read-file` job successfully printed "Hello from Job 1" - proving data was
shared between two separate job runners.

### When would you use artifacts in a real pipeline?

- Passing a compiled build output (e.g. a jar/binary/dist folder) from a "build" job to a "deploy" job.
- Saving test reports or coverage reports so they can be reviewed later, even after the CI machine is destroyed.
- Sharing generated files (like Docker image tarballs) between jobs that run on different runners.
- Keeping logs or diagnostic files for debugging failed runs, without needing to re-run the whole pipeline.

**Workflow code (two jobs):**
![Multi-job artifact code](Screenshots/day44-multi-job-artifact-code.png)

**Both jobs succeeded (create-file → read-file):**
![Multi-job success](Screenshots/day44-multi-job-success.png)

**Job 2 output - proof data was received from Job 1:**
![Multi-job output](Screenshots/day44-multi-job-output.png)

---

## Task 5: Run Real Tests in CI

Added a Python script (`hello.py`) with a simple assertion-based test to the repo root.

Created a workflow that:
- Checks out the code
- Sets up Python (3.11) using `actions/setup-python@v5`
- Runs the script using `python hello.py`

Verified the pipeline passes when the assertion is correct.

Then intentionally broke the test by changing the expected value in the `assert` statement.
The pipeline correctly failed with an `AssertionError`, showing the exact traceback and exit code 1.

Fixed the assertion back to the correct value and pushed again - pipeline turned green,
confirming the CI correctly reflects the health of the code.

**Workflow code:**
![Run tests workflow code](Screenshots/day44-run-tests-code.png)

**Workflow file reference (troubleshooting file path issue):**
![Workflow file reference](Screenshots/day44-workflow-file-reference.png)

**First successful run (after fixing file path issue):**
![Run tests original success](Screenshots/day44-run-tests-original-success.png)

**Confirmed passing run:**
![Run tests success](Screenshots/day44-run-tests-success.png)

**Pipeline failed after intentionally breaking the test:**
![Test failed intentionally](Screenshots/day44-test-failed-intentional.png)

**Failure logs showing AssertionError:**
![Test failure logs](Screenshots/day44-test-failure-logs.png)

**Script fixed back to correct value:**
![Hello.py fixed](Screenshots/day44-hello-py-fixed.png)

**Pipeline green again after fix:**
![Test fixed success](Screenshots/day44-test-fixed-success.png)

**Final logs - test passed:**
![Test final logs](Screenshots/day44-test-final-logs.png)

---

## Task 6: Caching

Added a workflow that installs a Python dependency (`requests`) using `pip install`.
Used `actions/setup-python@v5` with the built-in `cache: 'pip'` option to cache pip downloads.

First run: cache was saved (no prior cache existed).
Second run: pip logs showed "Using cached ..." for every package, confirming the
cached files were reused instead of being downloaded from the internet again -
the install step completed much faster (around 3 seconds).

### What is being cached and where is it stored?

- The pip download cache (wheel/package files downloaded by pip) is what gets cached -
  not the installed packages themselves, but the downloaded files pip uses to install them.
- GitHub Actions stores the cache in its own cache storage (tied to the repository),
  identified by a cache key. When a workflow runs again with a matching key, GitHub
  restores the cached files into the runner before the install step runs.
- If the dependency file or environment changes (different Python version, different
  packages), the cache key changes and a fresh cache is created instead of reusing the old one.

**Workflow code:**
![Cache workflow code](Screenshots/day44-cache-code.png)

**First run (cache created):**
![Cache first run](Screenshots/day44-cache-first-run.png)

**Cache successfully saved:**
![Cache saved success](Screenshots/day44-cache-saved-success.png)

**Cache hit proof - packages reused from cache:**
![Cache hit proof](Screenshots/day44-cache-hit-proof.png)

---

## Overall Learning: What I learned about secrets management

- Secrets must never be hardcoded in code or YAML files - they belong in GitHub's
  encrypted secrets vault, referenced only through `${{ secrets.NAME }}`.
- GitHub automatically masks any known secret value in logs, but this only works
  when the secret is referenced properly (directly or via an environment variable).
- Passing secrets through `env:` is the standard, safe pattern used by real CLI
  tools (Docker, AWS CLI, etc.) instead of passing them as raw command arguments.
- Artifacts and caching both solve the same underlying problem - CI runners are
  temporary and disposable - artifacts let you keep output files, while caching
  lets you skip repeating expensive work like dependency downloads.
- Running real tests with proper exit codes (via `assert` or a test framework) is
  what makes CI actually useful - without it, a "green" pipeline can be misleading.
