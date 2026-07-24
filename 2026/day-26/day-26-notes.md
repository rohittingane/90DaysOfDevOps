# Day 26 – GitHub CLI: Manage GitHub from Your Terminal

## Overview

Today I learned how to use GitHub CLI (`gh`) to manage GitHub repositories, issues, pull requests, and workflows directly from the terminal without depending on the GitHub web interface.

GitHub CLI is useful for DevOps engineers because it helps automate GitHub operations, manage repositories faster, and integrate GitHub tasks into scripts and CI/CD pipelines.

---

# Task 1: Install and Authenticate GitHub CLI

## Installation

Installed GitHub CLI on Ubuntu using:

```bash
sudo apt update
sudo apt install gh -y
```

Verified installation:

```bash
gh --version
```

![gh version](Screenshots/day26-task1-gh-version.png)

## Authentication

Authenticated GitHub account using:

```bash
gh auth login
```

Verified login:

```bash
gh auth status
```

Checked active GitHub user:

```bash
gh api user
```

![gh auth status](Screenshots/day26-task1-auth-status.png)

## Authentication Methods Supported by gh

GitHub CLI supports:

- Browser-based OAuth authentication
- Personal Access Token (PAT)
- GitHub Enterprise authentication
- SSH authentication

---

# Task 2: Working with Repositories

## Create Repository using gh

Created a GitHub repository directly from terminal:

```bash
gh repo create gh-cli-demo \
  --public \
  --description "GitHub CLI practice for Day 26" \
  --add-readme \
  --clone
```

This command:
- Created a public repository
- Added a README file
- Automatically cloned the repository locally

![repo created](Screenshots/day26-task2-repo-created.png)

## View Repository Details

```bash
gh repo view
gh repo view --json name,description,visibility,url,defaultBranchRef
```

## List Repositories

```bash
gh repo list
gh repo list rohittingane
```

## Open Repository

```bash
gh repo view --web
```

**Note:** EC2 server does not have a browser, so `xdg-open` failed. The command works normally on a local machine with browser support.

## Delete Repository

Added delete permission (default token scope doesn't include `delete_repo`):

```bash
gh auth refresh -h github.com -s delete_repo
```

Deleted repository:

```bash
gh repo delete gh-cli-demo --yes
```

![repo deleted](Screenshots/day26-task2-delete-repo.png)

---

# Task 3: Managing Issues

Created an issue using GitHub CLI:

```bash
gh issue create \
  --repo rohittingane/devops-git-practice \
  --title "Practice GitHub CLI Issues" \
  --body "Created this issue using GitHub CLI during Day 26 learning." \
  --label "documentation"
```

## List Issues

```bash
gh issue list --repo rohittingane/devops-git-practice
```

## View Issue

```bash
gh issue view 1 --repo rohittingane/devops-git-practice
```

## Close Issue

```bash
gh issue close 1 --repo rohittingane/devops-git-practice
```

Verified:

```bash
gh issue list --repo rohittingane/devops-git-practice
```

![issues flow](Screenshots/day26-task3-issues-flow.png)

## How can gh issue be used in automation?

`gh issue` can be used in scripts to automatically create issues when CI/CD pipelines fail, report bugs, track problems, and notify teams without manually opening GitHub.

---

# Task 4: Pull Requests using GitHub CLI

## Create Branch

```bash
git checkout -b gh-cli-pr-demo
```

Created a file:

```bash
echo "GitHub CLI Pull Request practice - Day 26" > gh-cli-pr-demo.txt
```

Committed changes:

```bash
git add gh-cli-pr-demo.txt
git commit -m "Add GitHub CLI PR practice file"
```

Pushed branch:

```bash
git push -u origin gh-cli-pr-demo
```

## Create Pull Request

```bash
gh pr create \
  --repo rohittingane/devops-git-practice \
  --title "GitHub CLI PR Practice - Day 26" \
  --body "Created this pull request using GitHub CLI from terminal."
```

![pr created](Screenshots/day26-task4-create-pr-view.png)

## List Pull Requests

```bash
gh pr list --repo rohittingane/devops-git-practice
```

## View Pull Request Details

```bash
gh pr view 2 \
  --repo rohittingane/devops-git-practice \
  --json title,state,author,reviewDecision,statusCheckRollup,mergeStateStatus
```

## Merge Pull Request

```bash
gh pr merge 2 \
  --repo rohittingane/devops-git-practice \
  --merge
```

Verified:

```bash
gh pr list --repo rohittingane/devops-git-practice
```

![pr merged](Screenshots/day26-task4-merge-pr.png)

## Supported Merge Methods

GitHub CLI supports:

**1. Merge Commit**
```bash
--merge
```
Creates a merge commit.

**2. Squash Merge**
```bash
--squash
```
Combines multiple commits into one commit.

**3. Rebase Merge**
```bash
--rebase
```
Replays commits on top of the target branch.

## Reviewing Someone Else's PR

A PR can be reviewed using:

```bash
gh pr list
gh pr view <number>
gh pr diff <number>
gh pr checks <number>
```

---

# Task 5: GitHub Actions & Workflows Preview

Used public repository: `cli/cli`

## List Workflow Runs

```bash
gh run list --repo cli/cli
```

![workflow list](Screenshots/day26-task5-workflow-list.png)

Learned:
- Workflow status
- Branch
- Trigger event
- Run ID
- Execution time

## View Workflow Run

```bash
gh run view 30117722104 --repo cli/cli
```

![workflow view](Screenshots/day26-task5-workflow-view.png)

Checked:
- Workflow name
- Jobs
- Status
- Execution details

## List Workflows

```bash
gh workflow list --repo cli/cli
```

## How gh run and gh workflow help in CI/CD?

`gh run` helps monitor GitHub Actions executions, check failures, and debug pipeline issues.

`gh workflow` helps manage automation workflows directly from the terminal.

---

# Task 6: Useful gh Tricks

*(No screenshots taken for this section — commands were explored directly and are documented below with their purpose and use cases.)*

## gh api

Used for making GitHub API calls:

```bash
gh api user
```

Use cases:
- Fetch GitHub user information
- Get repository data
- Use API responses in automation scripts

## gh gist

Used to manage GitHub Gists.

```bash
gh gist create
gh gist list
gh gist view
gh gist delete
```

Use cases:
- Share code snippets
- Store small scripts
- Manage snippets from terminal

## gh release

Used for managing GitHub releases.

```bash
gh release create
gh release list
gh release view
gh release upload
```

Use cases:
- Create software releases
- Upload release files
- Manage project versions

## gh alias

Used to create shortcuts.

```bash
gh alias set pv "pr view"
```

Now:

```bash
gh pv 2
```

works the same as:

```bash
gh pr view 2
```

Benefits:
- Saves time
- Creates shortcuts for frequently used commands

## gh search repos

Used to search GitHub repositories from terminal.

```bash
gh search repos kubernetes
```

Use cases:
- Find open-source projects
- Search repositories by technology
- Discover projects quickly

---

# Final Learning Summary

Today I learned how to manage GitHub completely from the terminal using GitHub CLI.

Key learnings:
- Repository management using `gh repo`
- Issue tracking using `gh issue`
- Pull request workflow using `gh pr`
- Monitoring GitHub Actions using `gh run` and `gh workflow`
- Using GitHub API and automation-friendly commands

GitHub CLI will help in DevOps automation by reducing manual GitHub operations and integrating repository management into scripts and CI/CD workflows.

