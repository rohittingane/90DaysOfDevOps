# Day 39 – What is CI/CD?

> Goal of today: Understand CI/CD so well that even a complete beginner reading this file can say "Oh, now I get why CI/CD exists, what it does, and how it works."

---

## 1. The Problem (Why CI/CD exists at all)

Imagine a team of **5 developers** working on the same project. Everyone writes code on their own laptop, and then someone manually copies/uploads the code to the production server.

### What goes wrong with manual deployment?

Think of it like 5 people cooking in the same kitchen, but there's no recipe, no checklist, and no one double-checking the food before it's served to customers.

- **Merge conflicts:** Two developers change the same file. When combined, the code breaks.
- **Human error:** A developer forgets one step while deploying manually (like setting an environment variable), and the whole website goes down.
- **No testing before deploy:** Someone pushes buggy code without testing it properly, and now the live app is broken — and nobody knows which change caused it.
- **No consistency:** Every developer deploys in their own way. One uses a different method than the other. There's no single standard process.
- **Slow and risky:** If something breaks, rolling back (undoing the change) manually takes time, during which real users are seeing a broken app.

**Real example:** Imagine an e-commerce site. A developer pushes a "quick fix" directly to production without testing. The payment page breaks. Now customers can't buy anything, and the team scrambles for an hour to find and fix the bug manually. This is exactly the kind of chaos CI/CD prevents.

### What does "It works on my machine" mean?

This is a famous phrase in software development. It means:

> The code runs perfectly fine on the developer's own laptop, but breaks when it runs somewhere else (like the production server).

**Why does this happen?**
The developer's laptop might have a different version of Node.js, different environment variables, different operating system, or different installed packages compared to the production server. So the *same code* behaves *differently* in different environments.

**Real example:** A developer builds a feature using Node.js version 20 on their laptop. The production server still runs Node.js version 16. Some new JavaScript feature the developer used doesn't exist in version 16 — so the app crashes in production, even though it worked "perfectly" on the developer's machine.

This is a **real problem** because:
- It's hard to detect until it's too late (in production, in front of real users).
- It wastes time — everyone starts debugging the "code" when the real issue is the "environment."

CI/CD solves this by running the code in the **exact same automated environment every single time** — so if it works in the pipeline, it will work in production too.

### How many times a day can a team safely deploy manually?

Realistically, a team deploying manually can only do it **1 to 2 times a day**, and even that requires a lot of careful manual checking, extra caution, and usually happens outside business hours (like late at night) to reduce risk.

Compare this to companies using CI/CD — for example, **Amazon deploys new code to production every single second**, because the entire process (build, test, deploy) is automated and doesn't rely on a human doing everything by hand.

---

## 2. CI vs CD vs CD (the three most confusing terms, explained simply)

These three terms sound similar but mean very different things. Think of them as three levels of automation — each one goes a bit further than the last.

### 🔹 Continuous Integration (CI)

**In simple words:** Every time a developer pushes code, the system automatically **builds the code and runs tests** to check if anything is broken.

- **What happens:** Code is pulled from the repository, dependencies are installed, and automated tests run.
- **How often:** Every single time someone pushes code or opens a pull request.
- **What it catches:** Bugs, broken code, failing tests — before the code gets merged into the main branch.

**Real-world example:** A developer at a company adds a new "Add to Cart" button. As soon as they push this code to GitHub, an automated pipeline runs unit tests to check if the cart logic still works correctly. If a test fails, the developer gets notified immediately — before their code even gets merged, let alone deployed.

### 🔹 Continuous Delivery

**In simple words:** After CI passes (code is built and tested), the code is automatically packaged and made **"ready to deploy" at any time** — but an actual human still has to click a button to send it live.

- **How it's different from CI:** CI stops at "tests passed." Continuous Delivery goes one step further — it prepares the final deployable package (like a Docker image or build artifact) automatically.
- **What "delivery" means:** The code is *delivered* to a ready state — sitting there, packaged and tested, waiting for someone to say "go live."

**Real-world example:** A team building a banking app. Every change is tested and automatically packaged into a deployable build, and pushed to a staging environment. But before it goes to real production (real money, real customers), a manager or lead engineer manually clicks "Approve & Deploy" — because they want a human check for something this critical.

### 🔹 Continuous Deployment

**In simple words:** Same as Continuous Delivery, but there's **no manual approval step at all**. If the tests pass, the code goes live automatically — no human clicks anything.

- **How it differs from Delivery:** Delivery = ready to deploy (human decides when). Deployment = actually deployed (fully automatic, no human involved).
- **When teams use it:** Teams that trust their automated tests completely, and want to ship changes to users as fast as possible — often many times a day.

**Real-world example:** A company like Meta (Facebook) — when a developer's code passes all automated checks, it can go live to real users within minutes, completely automatically, without anyone clicking "deploy."

### Quick Comparison Table

| Term | What happens | Manual step involved? |
|---|---|---|
| **Continuous Integration (CI)** | Code is built + tested automatically on every push | No manual step (just for build/test) |
| **Continuous Delivery** | Code is built, tested, AND packaged — ready to deploy | Yes — a human clicks "deploy" |
| **Continuous Deployment** | Code is built, tested, packaged, AND deployed | No — fully automatic |

**Easy way to remember:** CI is about catching bugs early. Delivery is "ready when you are." Deployment is "gone the moment it's ready."

---

## 3. Pipeline Anatomy (the building blocks of any CI/CD pipeline)

Think of a CI/CD pipeline like a factory assembly line. Raw material (your code) goes in one end, and a finished product (a working, deployed app) comes out the other end. Here are the parts of that assembly line:

| Part | What it does | Real example |
|---|---|---|
| **Trigger** | The event that starts the pipeline | A developer does `git push`, or opens a pull request |
| **Stage** | A big logical phase of the pipeline | "Build," "Test," "Deploy" are each a stage |
| **Job** | A specific task inside a stage | Inside the "Test" stage, a job could be "Run unit tests" |
| **Step** | One single command inside a job | Inside that job, a step could be `npm test` |
| **Runner** | The actual computer/machine that executes everything | A virtual machine (like `ubuntu-latest`) provided by GitHub Actions |
| **Artifact** | The output produced after a job finishes | A built `.zip` file, a Docker image, or a test report |

**How they connect:**
```
Trigger (push/PR)
   → Stage (e.g. Test)
       → Job (e.g. "Run eslint")
           → Steps (checkout code → install deps → run eslint command)
               → Runner (the machine that does all this work)
                   → Artifact (test report or build output)
```

---

## 4. Pipeline Diagram

**Scenario:** A developer pushes code to GitHub → app gets tested → built into a Docker image → deployed to a staging server.

```
[Developer Push to GitHub]
            │
            ▼
   ┌─────────────────┐
   │   STAGE 1        │
   │   BUILD          │  → install dependencies, compile code
   └─────────────────┘
            │
            ▼
   ┌─────────────────┐
   │   STAGE 2        │
   │   TEST           │  → run unit tests + integration tests
   └─────────────────┘
            │
            ▼
   ┌─────────────────┐
   │   STAGE 3        │
   │   DOCKER BUILD   │  → package app into a Docker image (this is the artifact)
   └─────────────────┘
            │
            ▼
   ┌─────────────────┐
   │   STAGE 4        │
   │   DEPLOY         │  → push Docker image to staging server
   └─────────────────┘
            │
            ▼
   [✅ App Live on Staging]
```

*(A visual version of this diagram was also generated and can be screenshotted for the LinkedIn "Learn in Public" post.)*

### End-to-end flow: Developer to End User (horizontal)

This extends the diagram above to show the **complete journey** — starting from the developer writing code, all the way to a real user opening the app.

```
[Developer]──▶[GitHub]──▶[Test (CI)]──▶[Build]
   Push code    Repo       Run tests    Docker image
   to repo      trigger                      │
                                              ▼
[Staging]──▶[Production]──▶[End user]
 QA verifies   Live for       Opens app
 the build     real users     on device
```

**Flow explained in one line:**
`Developer → GitHub → Test (CI) → Build → Staging → Production → End user`

- **Developer → GitHub:** Code is pushed, which triggers the pipeline.
- **Test (CI):** Automated tests run to catch bugs early.
- **Build:** App is packaged into a Docker image (the artifact).
- **Staging:** The image is deployed to a staging server where the QA team verifies everything works.
- **Production:** Once approved, the same image is deployed to the live production server.
- **End user:** A real person opens the app on their browser or phone — and this is the final outcome the entire pipeline exists to deliver, safely and repeatedly.

*(A visual horizontal version of this end-to-end flow was also generated and can be screenshotted for the LinkedIn "Learn in Public" post.)*

---

## 5. Explore in the Wild — Real Workflow from an Open-Source Repo

**Repo explored:** `facebook/react` — file: `.github/workflows/lint.yml`

This is a real, production CI workflow used by one of the most popular open-source projects in the world. Let's break it down like a beginner would understand it.

### What triggers it?
- Runs on every `push` to the **`main`** branch.
- Runs on **every pull request**, no matter which branch it targets.
- Uses a `concurrency` setting — if a new push comes in while an old run is still going, the old run is **automatically cancelled**. This saves time and computing resources (no point testing outdated code).

### How many jobs does it have?
**4 jobs**, and they all run **in parallel** (at the same time, independently of each other):

1. **`prettier`** — checks if the code formatting follows the team's style rules (like spacing, quotes, etc.)
2. **`eslint`** — checks code quality and catches common mistakes or bad patterns
3. **`check_license`** — makes sure every file has the correct license header (legal requirement for open-source)
4. **`test_print_warnings`** — checks that console warning messages in React work correctly

### What does it do (in simple words)?

This workflow is purely a **"quality gate."** It doesn't build a final app or deploy anything — its only job is to make sure the code someone is trying to merge:
- Is formatted correctly
- Follows code quality rules
- Has proper license headers
- Doesn't break how warnings are printed

Every job follows the same basic pattern (steps):
1. **Checkout the code** — get the latest code from the repo
2. **Set up Node.js** — install the correct Node version (from `.nvmrc` file)
3. **Cache `node_modules`** — save installed packages so future runs are faster (don't reinstall everything every time)
4. **Install dependencies** — only if the cache didn't already have them
5. **Run the actual check** — e.g., `yarn prettier-check`, `eslint`, license check script, etc.

### Interesting real-world detail
The workflow sets `permissions: {}` — meaning it is given **zero extra GitHub permissions**. This is a security best practice: a workflow that only checks code (linting) doesn't need permission to modify anything in the repo, so React's maintainers deliberately locked it down to the bare minimum.

**Why this matters for beginners:** This shows CI in action — every single PR to React gets automatically checked for formatting, code quality, and license issues **before a human even reviews it**. This saves reviewers a huge amount of time because they don't have to manually check for style mistakes — the pipeline already did it.

---

## Key Takeaways (in one line each)

- CI/CD is a **practice/process**, not a single tool. GitHub Actions, Jenkins, GitLab CI, CircleCI are just tools that help you implement it.
- **CI** = automatically build + test code on every push (catch bugs early).
- **Continuous Delivery** = code is always ready to deploy, but a human presses the final button.
- **Continuous Deployment** = code goes live automatically, no human needed.
- A pipeline is made of: **Trigger → Stage → Job → Step → Runner → Artifact**.
- **A pipeline failing is not a bad thing** — it's the pipeline doing exactly what it's supposed to do: catching a problem before it reaches real users.

