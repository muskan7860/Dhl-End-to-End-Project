# DAY 3 — Git & GitHub for DevOps

> **Project:** DHL-inspired Enterprise Logistics & Shipment Tracking
> Platform  
> **Goal:** Understand Git deeply enough to use it safely in CI/CD,
> GitOps, incident recovery, and team collaboration.
>
> **Core rule:** **Git is not only a place to store code. It is the
> change-control system behind modern DevOps workflows.**

------------------------------------------------------------------------

# 1. Why Git Matters in DevOps

In our project, Git will control:

``` text
Application Code
Jenkinsfile
Dockerfiles
Terraform
Kubernetes YAML
GitOps manifests
Documentation
```

This means one wrong Git action can affect:

``` text
Build
Deployment
Infrastructure
Production Configuration
Rollback
Audit Trail
```

------------------------------------------------------------------------

# 2. Git vs GitHub

## Git

Git is the distributed version-control system running locally.

It tracks:

``` text
Files
Changes
Commits
Branches
Tags
History
```

## GitHub

GitHub hosts Git repositories remotely and adds collaboration features:

``` text
Pull Requests
Reviews
Branch Protection
Issues
Actions
Security
Access Control
Auditability
```

------------------------------------------------------------------------

# 3. Git Mental Model

Think in four areas:

``` text
Working Directory
      ↓ git add
Staging Area
      ↓ git commit
Local Repository
      ↓ git push
Remote Repository
```

## Working Directory

Files you are currently editing.

Check:

``` bash
git status
```

## Staging Area

Changes selected for the next commit.

``` bash
git add <file>
```

## Local Repository

Committed history on your machine.

``` bash
git commit -m "message"
```

## Remote Repository

Shared repository such as GitHub.

``` bash
git push
```

------------------------------------------------------------------------

# 4. Repository Validation

For the DHL project:

``` bash
cd ~/dhl
git status
git remote -v
git branch
git log --oneline --decorate -10
```

These commands answer:

``` text
Am I inside a Git repo?
Which branch am I on?
What changed?
Which remote is configured?
What recent commits exist?
```

------------------------------------------------------------------------

# 5. Clone vs Fork

## Clone

Copies a repository to your local machine.

``` bash
git clone <repository-url>
```

## Fork

Creates your own GitHub-side copy of another repository.

Our DHL repository is a fork of the mentor/reference repository.

Conceptually:

``` text
Upstream Repository
        ↓ Fork
Your GitHub Repository
        ↓ Clone
Your Laptop
```

------------------------------------------------------------------------

# 6. origin vs upstream

Typical fork workflow:

``` text
origin
→ your repository

upstream
→ original/reference repository
```

Check:

``` bash
git remote -v
```

Add upstream if required:

``` bash
git remote add upstream <upstream-url>
```

Fetch it:

``` bash
git fetch upstream
```

Important:

> `git fetch` downloads remote history but does not automatically modify
> your current working branch.

------------------------------------------------------------------------

# 7. Branches

Branches isolate work.

Example:

``` text
main
 ├── feature/docker-backend
 ├── feature/jenkins-ci
 ├── feature/terraform-vpc
 └── fix/backend-port
```

Create:

``` bash
git switch -c feature/day3-git-docs
```

Older equivalent:

``` bash
git checkout -b feature/day3-git-docs
```

Check:

``` bash
git branch
```

------------------------------------------------------------------------

# 8. Why DevOps Should Not Work Directly on main

A protected/default branch should represent reviewed, validated code.

Recommended flow:

``` text
main
 ↓
feature branch
 ↓
changes
 ↓
commit
 ↓
push
 ↓
Pull Request
 ↓
CI checks
 ↓
review
 ↓
merge
```

This becomes extremely important when changes involve:

``` text
Jenkinsfile
Terraform
Kubernetes
GitOps
Security policies
```

------------------------------------------------------------------------

# 9. Commit Design

A commit should represent one logical change.

Good:

``` text
feat: add backend Dockerfile
fix: correct backend service port
docs: add Day 3 Git notes
chore: update dependency lockfile
```

Weak:

``` text
update
changes
final
working now
```

Why?

Good commits improve:

``` text
Review
Rollback
git bisect
Audit
Troubleshooting
```

------------------------------------------------------------------------

# 10. Staging Carefully

Avoid blindly running:

``` bash
git add .
```

when you do not understand what changed.

First:

``` bash
git status
git diff
```

Then:

``` bash
git add <specific-file>
```

Inspect staged changes:

``` bash
git diff --staged
```

Then commit.

------------------------------------------------------------------------

# 11. git diff

Unstaged:

``` bash
git diff
```

Staged:

``` bash
git diff --staged
```

Between commits:

``` bash
git diff <commit1>..<commit2>
```

DevOps use:

``` text
Which Terraform changed?
Which Kubernetes field changed?
Which image tag changed?
Which Jenkins stage changed?
```

------------------------------------------------------------------------

# 12. Push and Tracking

First push:

``` bash
git push -u origin feature/day3-git-docs
```

`-u` sets the upstream tracking branch.

Later:

``` bash
git push
```

------------------------------------------------------------------------

# 13. Pull Requests

A Pull Request is not just “please merge my code.”

It is a control point for:

``` text
Review
CI
Security Checks
Discussion
Approval
Change History
```

For production-impacting repositories, PRs should be part of change
control.

------------------------------------------------------------------------

# 14. Branch Protection

Important branches such as `main` can be protected.

Typical controls:

``` text
Require Pull Request
Require Review
Require Status Checks
Block Force Push
Block Deletion
Require Conversation Resolution
```

This protects the source of truth.

------------------------------------------------------------------------

# 15. Merge

Suppose:

``` text
main:       A---B
                 \
feature:          C---D
```

A merge may create:

``` text
A---B-------M
     \     /
      C---D
```

Merge preserves the branch relationship.

Command:

``` bash
git merge feature-name
```

------------------------------------------------------------------------

# 16. Rebase

Rebase rewrites the base of your branch.

Before:

``` text
main: A---B---C
          \
feature:   D---E
```

After rebase:

``` text
main: A---B---C
              \
               D'---E'
```

Commands:

``` bash
git switch feature
git fetch origin
git rebase origin/main
```

Important:

> Rebase rewrites commit history. Be careful rebasing shared branches.

------------------------------------------------------------------------

# 17. Merge vs Rebase

| Merge                    | Rebase                              |
|--------------------------|-------------------------------------|
| Preserves branch history | Produces linear history             |
| May create merge commit  | Rewrites commits                    |
| Safe for shared history  | Be careful on shared branches       |
| Useful for integration   | Useful for cleaning feature history |

Interview rule:

> Do not say one is always better. Explain the team's workflow and
> history requirements.

------------------------------------------------------------------------

# 18. Merge Conflicts

Conflict example:

``` text
main changed line X
feature changed same line X
```

Git cannot decide automatically.

Workflow:

``` bash
git status
```

Open conflicted file:

``` text
<<<<<<< HEAD
current branch content
=======
incoming content
>>>>>>> other-branch
```

Resolve manually, then:

``` bash
git add <file>
git commit
```

For rebase:

``` bash
git add <file>
git rebase --continue
```

Abort if needed:

``` bash
git merge --abort
```

or:

``` bash
git rebase --abort
```

------------------------------------------------------------------------

# 19. git fetch vs git pull

## fetch

``` bash
git fetch origin
```

Downloads references/history.

Does not automatically merge into current branch.

## pull

Conceptually:

``` text
git pull
≈
git fetch
+
integration step
```

Depending on configuration, integration can be merge or rebase.

For controlled troubleshooting, `fetch` is often safer because it lets
you inspect before integrating.

------------------------------------------------------------------------

# 20. git restore

Discard unstaged changes to a tracked file:

``` bash
git restore <file>
```

Unstage a staged file:

``` bash
git restore --staged <file>
```

Be careful: discarding local modifications can destroy uncommitted work.

------------------------------------------------------------------------

# 21. git reset

`reset` moves branch/index state.

Examples:

``` bash
git reset --soft HEAD~1
```

Moves HEAD back but keeps changes staged.

``` bash
git reset --mixed HEAD~1
```

Moves HEAD back and leaves changes unstaged.

``` bash
git reset --hard HEAD~1
```

Moves HEAD back and discards tracked working-tree changes.

**High risk:** `--hard` can destroy local work.

------------------------------------------------------------------------

# 22. git revert

For shared history, revert is usually safer.

``` bash
git revert <commit>
```

It creates a **new commit** that reverses an earlier commit.

Concept:

``` text
Bad commit exists in history
        ↓
Revert commit
        ↓
History stays auditable
```

------------------------------------------------------------------------

# 23. reset vs revert

| reset                 | revert                                 |
|-----------------------|----------------------------------------|
| Moves history pointer | Adds inverse commit                    |
| Can rewrite history   | Preserves shared history               |
| Useful locally        | Preferred for shared/protected history |
| `--hard` destructive  | Auditable                              |

Production rule:

> If a bad change is already shared/merged, prefer `revert` unless there
> is a very specific reason to rewrite history.

------------------------------------------------------------------------

# 24. git stash

Temporarily save uncommitted work:

``` bash
git stash
```

List:

``` bash
git stash list
```

Restore:

``` bash
git stash pop
```

Use case:

``` text
You are editing Terraform
Urgent production fix arrives
→ stash unfinished work
→ switch branch
→ fix incident
→ return
→ restore stash
```

------------------------------------------------------------------------

# 25. Tags

Tags mark important commits.

Example:

``` bash
git tag v1.0.0
git push origin v1.0.0
```

Annotated:

``` bash
git tag -a v1.0.0 -m "Release v1.0.0"
```

Use:

``` text
Release identification
Audit
Rollback reference
Artifact correlation
```

------------------------------------------------------------------------

# 26. Commit SHA

Every commit has an identifier.

``` bash
git log --oneline
```

Example:

``` text
a1b2c3d fix backend service port
```

In CI/CD, commit SHA can be used as:

``` text
Docker image tag
Deployment reference
Release traceability
```

Example:

``` text
ECR image:
dhl-backend:a1b2c3d
```

That is much more traceable than only using:

``` text
latest
```

------------------------------------------------------------------------

# 27. Git and Jenkins

Future Jenkins flow:

``` text
Developer Push
     ↓
GitHub
     ↓ webhook
Jenkins
     ↓
Checkout exact commit
     ↓
Build/Test/Scan
     ↓
Image tagged with commit/version
```

The pipeline must know exactly which Git revision it built.

------------------------------------------------------------------------

# 28. Git and GitOps

Later:

``` text
Application Repo
     ↓ Jenkins CI
ECR Image
     ↓
GitOps Repo image tag update
     ↓
Commit
     ↓
Argo CD detects Git change
     ↓
EKS reconciliation
```

Git becomes the desired-state audit trail.

------------------------------------------------------------------------

# 29. Application Repo vs GitOps Repo

Recommended separation:

``` text
Application Repository
├── source
├── tests
├── Dockerfile
└── Jenkinsfile

GitOps Repository
├── deployment manifests
├── environment overlays
└── image versions
```

Why?

It separates:

``` text
Code lifecycle
from
Deployment-state lifecycle
```

------------------------------------------------------------------------

# 30. DevOps Git Troubleshooting

## Problem: `git push` rejected

Possible causes:

``` text
Remote has newer commits
Branch protection
Permissions
Non-fast-forward update
```

Investigate:

``` bash
git status
git branch -vv
git fetch origin
git log --oneline --graph --decorate --all -20
```

Do not immediately force push.

------------------------------------------------------------------------

# 31. Problem: Accidentally committed secret

Deleting the line in the next commit is **not enough**.

Why?

The secret may still exist in Git history and external clones/caches.

Immediate response:

``` text
1. Treat secret as compromised
2. Rotate/revoke it
3. Remove it from current source
4. Evaluate history cleanup
5. Add prevention
6. Review access/logs where relevant
```

Never solve a secret leak by only doing:

``` bash
git rm secret.txt
git commit
```

Rotation is the critical first security action.

------------------------------------------------------------------------

# 32. Problem: Bad commit merged to main

Preferred controlled response:

``` bash
git log --oneline
git revert <bad-commit>
git push through normal protected workflow
```

Then investigate root cause.

------------------------------------------------------------------------

# 33. Problem: Merge conflict in Kubernetes YAML

Do not accept “ours” or “theirs” blindly.

Check:

``` text
Which image tag is intended?
Which environment is affected?
Which resource field changed?
Did both branches add valid requirements?
```

Validate after resolving:

``` bash
kubectl apply --dry-run=client -f <file>
```

or appropriate validation tooling in the later Kubernetes phase.

------------------------------------------------------------------------

# 34. Problem: Local branch is behind main

Safe pattern:

``` bash
git status
git fetch origin
git log --oneline --graph --decorate --all -20
```

Then choose team-approved merge/rebase strategy.

Do not copy commands blindly.

------------------------------------------------------------------------

# 35. Hands-On Lab for DHL Repo

Run:

``` bash
cd ~/dhl

git status

git remote -v

git branch -vv

git log --oneline --decorate -10
```

Create training branch:

``` bash
git switch -c docs/day3-git-training
```

Create a small doc:

``` bash
mkdir -p docs
printf "# Day 3 Git Training\n" > docs/DAY-03-TEST.md
```

Inspect:

``` bash
git status
git diff
```

Stage:

``` bash
git add docs/DAY-03-TEST.md
```

Inspect staged content:

``` bash
git diff --staged
```

Commit:

``` bash
git commit -m "docs: add Day 3 Git training note"
```

Inspect history:

``` bash
git log --oneline --decorate -5
```

Push:

``` bash
git push -u origin docs/day3-git-training
```

Then create a Pull Request in GitHub rather than directly modifying
`main`.

------------------------------------------------------------------------

# 36. Failure Lab — Wrong Commit

Create a safe test commit, then study:

``` bash
git log --oneline
```

If it is **local and not shared**, compare:

``` bash
git reset --soft HEAD~1
```

with:

``` bash
git reset --mixed HEAD~1
```

Do not practice `--hard` with valuable uncommitted work.

------------------------------------------------------------------------

# 37. Failure Lab — Shared Bad Commit

Simulate on a training branch:

``` bash
git revert <commit>
```

Observe:

``` text
Original commit remains
New inverse commit appears
```

This is why revert is safer for shared history.

------------------------------------------------------------------------

# 38. Failure Lab — Merge Conflict

Create two branches that edit the same line.

Then merge them.

When conflict appears:

``` bash
git status
```

Resolve the markers manually.

Then:

``` bash
git add <file>
git commit
```

The objective is to understand conflict resolution, not memorize
commands.

------------------------------------------------------------------------

# 39. Production Git Controls

For important repositories:

``` text
Protected main
Required Pull Request
Required Review
Required CI Checks
No routine force push
Least-privilege access
Secret scanning
CODEOWNERS where useful
Signed commits where required
Audit-friendly history
```

------------------------------------------------------------------------

# 40. Day 3 Deployment Connection

What we learn today directly affects later phases:

| Git Concept       | Later Use                             |
|-------------------|---------------------------------------|
| Branch            | Feature/change isolation              |
| PR                | CI and review gate                    |
| Commit SHA        | Docker/ECR traceability               |
| Tag               | Release                               |
| Revert            | Rollback/change recovery              |
| Branch protection | Production source-of-truth protection |
| GitOps commit     | Argo CD deployment trigger            |
| Diff              | Review Terraform/K8s changes          |
| History           | Incident investigation                |

------------------------------------------------------------------------

# 41. Day 3 Hands-On Questions

For every Git command, ask:

``` text
What state does this command read/change?
Does it affect working tree, staging area, local history or remote history?
Can it destroy work?
Does it rewrite shared history?
How would this affect CI/CD?
```

------------------------------------------------------------------------

# 42. Completion Checklist

- [ ] Explain Git vs GitHub
- [ ] Explain working tree/staging/local/remote
- [ ] Clone and inspect a repository
- [ ] Explain origin vs upstream
- [ ] Create feature branches
- [ ] Stage specific changes
- [ ] Read `git diff`
- [ ] Create meaningful commits
- [ ] Push and create PRs
- [ ] Explain branch protection
- [ ] Explain merge vs rebase
- [ ] Resolve merge conflicts
- [ ] Explain fetch vs pull
- [ ] Explain restore/reset/revert
- [ ] Use stash safely
- [ ] Explain tags and commit SHA
- [ ] Explain Git in Jenkins
- [ ] Explain Git in GitOps
- [ ] Recover from a bad shared commit
- [ ] Explain secret-leak response
- [ ] Avoid unsafe force pushes

# Day 3 Golden Rule

> **Before running a Git recovery command, know which state it changes
> and whether the history is already shared.**
