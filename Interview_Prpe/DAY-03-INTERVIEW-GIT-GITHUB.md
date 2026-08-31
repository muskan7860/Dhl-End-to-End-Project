# DAY 3 — Git & GitHub Interview Training

> **Target:** DevOps Engineer — approximately 4 years experience.

## Basic

### Q1. Git vs GitHub?

> Git is the distributed version-control system that tracks repository
> history locally. GitHub hosts Git repositories and adds collaboration
> controls such as pull requests, reviews, branch protection,
> permissions and CI integrations.

### Q2. Explain Git's working areas.

> Working directory contains active edits, staging contains changes
> selected for the next commit, the local repository contains committed
> history, and the remote repository is the shared server-side copy such
> as GitHub.

### Q3. What does `git add` do?

> It updates the staging/index state for the next commit. It does not
> push anything to GitHub.

### Q4. What does `git commit` do?

> It creates a new local commit containing the staged snapshot and
> metadata.

### Q5. What does `git push` do?

> It transfers local Git references/commits to the configured remote
> subject to permissions and branch rules.

## Normal / Practical

### Q6. Why use feature branches?

> They isolate changes from the protected/default branch and allow
> review and CI before integration.

### Q7. `git fetch` vs `git pull`?

> Fetch downloads remote history without automatically integrating it.
> Pull fetches and then integrates according to configuration, typically
> merge or rebase.

### Q8. Why inspect `git diff` before commit?

> It verifies exactly what changed and prevents accidental
> configuration, secret or unrelated changes from entering a commit.

### Q9. origin vs upstream?

> In a fork workflow, origin typically points to my fork and upstream
> points to the original source repository.

### Q10. Why meaningful commit messages?

> They improve review, auditability, incident investigation, rollback
> selection and history search.

## Mid-Level

### Q11. Merge vs rebase?

> Merge integrates histories and may create a merge commit. Rebase
> reapplies commits onto a new base and rewrites those commits. I use
> the team's defined workflow and avoid rebasing shared history
> carelessly.

### Q12. reset vs revert?

> Reset moves branch/index state and can rewrite history. Revert creates
> a new commit that inverses an earlier commit, so it is generally safer
> for changes already shared or merged.

### Q13. What is `git reset --hard` risk?

> It can discard tracked working-tree/index changes and move history. I
> never use it blindly when valuable uncommitted work exists.

### Q14. What is stash?

> A temporary place for uncommitted work. Useful when I need to switch
> context without committing incomplete changes.

### Q15. Why tags?

> Tags identify important commits such as releases and provide stable
> references for release traceability.

## Advanced

### Q16. Why use commit SHA as image tag?

> It provides direct traceability from the deployed container artifact
> back to the exact source revision Jenkins built. It is more
> deterministic than relying only on a mutable `latest` tag.

### Q17. Why protect main?

> Because main often represents releasable or production-impacting
> state. Protection can enforce pull requests, reviews, CI status checks
> and restrictions on force pushes/deletion.

### Q18. Why is force push dangerous?

> It rewrites remote history and can remove commits other people or
> systems depend on. On protected/shared branches it should normally be
> blocked.

### Q19. What if a secret is committed?

> I treat the credential as compromised and rotate/revoke it
> immediately. Then remove it from current code, assess history cleanup
> and add prevention. Deleting it in a later commit does not remove
> prior exposure.

### Q20. What is a detached HEAD?

> HEAD points directly at a commit rather than a branch. New commits can
> become difficult to retain unless I create or move a branch reference.

## Architecture / DevOps

### Q21. How does Git interact with Jenkins?

> GitHub stores source and pipeline configuration. A push or PR can
> trigger Jenkins, which checks out a specific revision,
> builds/tests/scans it and produces an artifact traceable to that
> commit.

### Q22. How does Git interact with Argo CD?

> The GitOps repository stores desired Kubernetes state. Argo CD
> compares that Git state with the live cluster and reconciles changes
> according to policy.

### Q23. Why separate application and GitOps repositories?

> It separates application-source lifecycle from deployment-state
> lifecycle and gives clearer permissions, audit boundaries and
> deployment history.

### Q24. What should be source of truth for deployment?

> In our target GitOps model, the GitOps repository is the desired-state
> source of truth for Kubernetes deployment configuration.

## Troubleshooting

### Q25. `git push` rejected: non-fast-forward. What do you do?

> I do not force push immediately. I run `git status`, fetch the remote,
> inspect branch divergence with graph/log commands, then integrate
> according to the team's merge/rebase policy and push again.

### Q26. Merge conflict in Kubernetes YAML?

> I inspect both intended changes instead of choosing ours/theirs
> blindly, resolve the semantic configuration correctly, then validate
> YAML/Kubernetes configuration before committing.

### Q27. Bad commit merged to main?

> Prefer a controlled revert through the protected workflow. Revert
> preserves the audit trail while restoring the previous behavior.

### Q28. Accidentally deleted local changes?

> Recovery depends on whether changes were committed, staged, stashed or
> purely untracked/uncommitted. I first inspect Git state and reflog
> where appropriate rather than running additional destructive commands.

## Scenarios

### Scenario 1 — Production broke after a merge

> I correlate the incident with the exact deployment and commit SHA,
> inspect the diff, determine whether rollback/revert is the safest
> mitigation, restore service first through the approved workflow, then
> investigate why CI/review did not catch it.

### Scenario 2 — Developer force-pushed main

> I identify the previous branch tip using available remote/audit/reflog
> references, stop further writes if needed, restore the intended
> history carefully, and strengthen branch protection to prevent
> recurrence.

### Scenario 3 — Terraform change and app change mixed in one commit

> I would prefer logically separate commits/PRs because infrastructure
> and application changes may have different reviewers, risks and
> rollback paths. If already submitted, I may ask to split the change
> before merge.

### Scenario 4 — GitOps repo has wrong image tag

> Argo CD may correctly deploy the wrong desired state. I identify the
> Git commit that changed the tag, verify the intended ECR artifact,
> correct or revert the GitOps commit, then let Argo CD reconcile.

## Project Answer

### Q29. How are you using Git in your DHL project?

> In my hands-on enterprise-style DHL-inspired project, Git is the
> change-control foundation. Application source, Jenkins configuration
> and documentation live in the application repository, while later I
> will maintain deployment state through GitOps. I use feature branches
> and pull requests for controlled changes, commit SHAs for artifact
> traceability, and revert rather than rewriting shared history when
> recovering from bad merged changes.

## Cross-Questions

- What is HEAD?
- What is a remote-tracking branch?
- `git checkout` vs `git switch`?
- `git restore` vs `git reset`?
- What is reflog?
- What is cherry-pick?
- When would you squash commits?
- What is fast-forward merge?
- What is a merge commit?
- Why not use `latest` as deployment version?
- Why should main require CI checks?
- How do branch protections affect Jenkins?
- How do you roll back GitOps?
- What happens if Git and live cluster differ?

## STAR

**Situation:** A multi-service application required a controlled
delivery workflow.

**Task:** Establish safe source-control practices before CI/CD and
GitOps automation.

**Action:** I used feature branches, meaningful commits, diffs and
pull-request-based integration; designed protected-main practices; used
immutable Git revisions for artifact traceability; and defined
revert-based recovery for bad shared changes.

**Result:** The repository becomes auditable and CI/CD-friendly, and
every build/deployment can be traced back to an approved source
revision.

## Rapid Fire

1.  Git vs GitHub?
2.  Working tree?
3.  Staging area?
4.  Commit?
5.  Branch?
6.  origin?
7.  upstream?
8.  fetch vs pull?
9.  merge vs rebase?
10. reset vs revert?
11. restore?
12. stash?
13. tag?
14. commit SHA?
15. branch protection?
16. PR?
17. force-push risk?
18. secret committed?
19. Git in Jenkins?
20. Git in GitOps?

## Completion Checklist

- [ ] Explain Git internal workflow
- [ ] Explain fork/clone/origin/upstream
- [ ] Create and manage branches
- [ ] Inspect diffs before commits
- [ ] Explain PR and branch protection
- [ ] Explain merge/rebase
- [ ] Explain reset/revert
- [ ] Resolve merge conflicts
- [ ] Explain tags and SHA traceability
- [ ] Troubleshoot rejected pushes
- [ ] Handle bad merged commits
- [ ] Explain secret-leak response
- [ ] Explain Git in CI/CD and GitOps
- [ ] Give project answer naturally

# Incident Framework

``` text
Identify exact commit/change
→ establish blast radius
→ inspect diff/history
→ choose non-destructive recovery
→ restore service
→ validate
→ root cause
→ prevention
```
