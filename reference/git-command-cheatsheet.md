# Git Command Cheatsheet For OpenShift And VS Code

Author: Emmanuel Naweji
Role: PhD Candidate at National University; Infrastructure, Cloud, AI, and DevOps Engineer; Mentor

## Purpose

This cheatsheet focuses on the Git commands OpenShift engineers use most often when working with cluster configuration, GitOps repositories, and VS Code.

## How Beginners Should Use This Cheatsheet

If you are new to Git, do not try to memorize every command at once. Use this cheatsheet in this order:

1. check what changed
2. create or switch to the correct branch
3. stage only the files you intended to change
4. create a clear commit
5. push your branch

Simple meaning of common Git words:

- `repository` means the project folder tracked by Git
- `branch` means your working line of changes
- `commit` means a saved checkpoint with a message
- `push` means sending your commits to the remote Git server
- `pull` means bringing remote changes down to your machine
- `remote` usually means GitHub or another central Git server

## Core Status And History

```bash
git status
git branch --show-current
git log --oneline --decorate --graph -20
git diff
git diff --staged
```

Use these to answer three questions first:

1. What branch am I on?
2. What changed?
3. What am I about to commit?

Beginner explanation:

- `git status` is the safest command to run first because it shows where you are and what is modified
- `git branch --show-current` tells you the active branch so you do not commit to the wrong place
- `git log` shows earlier commits and helps you understand recent history
- `git diff` shows exactly what changed in your files before you commit

## Branching

```bash
git checkout -b feature/ocp-lab-01
git switch main
git switch -c fix/storage-validation
git fetch origin
git pull --ff-only
```

Recommended naming patterns:

- `feature/<cluster-or-topic>`
- `fix/<issue-or-component>`
- `docs/<runbook-or-cheatsheet>`

Beginner explanation:

- create a branch before making changes so your work stays isolated
- `git checkout -b ...` or `git switch -c ...` creates a new branch and moves you into it
- `git fetch origin` updates your local view of the remote repository without changing your files
- `git pull --ff-only` updates your branch safely when no merge commit is needed

## Staging And Commit Hygiene

```bash
git add <file>
git add .
git restore --staged <file>
git commit -m "Add storage validation VM examples"
```

Avoid broad commits when working on multiple OpenShift clusters or environments at once.

Beginner explanation:

- `git add <file>` stages one file for the next commit
- `git add .` stages everything in the current folder, which is faster but easier to misuse
- `git restore --staged <file>` removes a file from the staged set without deleting your work
- `git commit -m ...` creates a checkpoint with a message that should explain what changed

Good beginner habit:

- prefer `git add <file>` over `git add .` until you are comfortable reviewing diffs

## Remote Operations

```bash
git remote -v
git push origin feature/ocp-lab-01
git fetch --all --prune
git pull --ff-only origin main
```

Beginner explanation:

- `git remote -v` shows where your repository pushes and pulls from
- `git push` sends your branch to the server
- `git fetch --all --prune` refreshes your view of remote branches and removes stale references
- `git pull --ff-only origin main` updates your local branch from `main` without creating an unnecessary merge commit

## Useful Review Commands For GitOps Repos

```bash
git diff main...HEAD
git show --stat
git log -- path/to/file.yaml
git blame path/to/file.yaml
```

These help you review who changed a cluster definition, when it changed, and whether your branch diverged cleanly from main.

Beginner explanation:

- use these commands before pushing YAML that affects clusters
- `git blame` is especially useful when you need to know who last changed a line and why
- `git log -- path/to/file.yaml` is useful when you only care about one file, not the whole repo

## Safe Undo Patterns

```bash
git restore <file>
git restore --source=HEAD~1 <file>
git revert <commit-sha>
```

Preferred guidance:

- use `git restore` for local unstaged file changes
- use `git revert` for committed history in shared branches
- avoid history rewrites on shared GitOps branches unless your team explicitly permits it

Beginner explanation:

- `git restore <file>` discards local changes in a file that you have not committed
- `git restore --source=HEAD~1 <file>` replaces a file with the version from the previous commit
- `git revert <commit-sha>` creates a new commit that undoes an older one; this is safer than rewriting shared history

## Stash When You Need To Pivot

```bash
git stash push -m "wip storage vlan updates"
git stash list
git stash pop
```

Do not leave long-lived stashes holding critical cluster changes.

Beginner explanation:

- `git stash` is a temporary shelf for uncommitted work
- use it when you must switch tasks quickly but are not ready to commit
- return to stashed work soon so it does not get forgotten

## OpenShift-Focused Git Workflow

Typical safe workflow for cluster configuration:

```bash
git checkout -b feature/ocp-lab-01
git add helm-charts/cluster-install/ocp-lab-01.yaml
git commit -m "Add sanitized config for ocp-lab-01"
helm template ./helm-charts/cluster-install -f ./helm-charts/cluster-install/ocp-lab-01.yaml
git push origin feature/ocp-lab-01
```

Recommended pairing before push:

- `yamllint`
- `helm template`
- `oc apply --dry-run=client -f <file>`

Beginner explanation:

- OpenShift and GitOps repos often contain YAML that can break deployments if committed carelessly
- the pattern above means: make a branch, stage the right file, commit it, validate it, then push
- validation commands reduce the chance of shipping broken manifests into Argo CD or cluster automation

## VS Code Source Control Tips

Inside VS Code you can:

- review changed files in the Source Control panel
- stage specific hunks instead of entire files
- open the integrated terminal and run the same Git commands listed here
- compare your working tree against the last commit from the editor

Useful command palette items:

```text
Git: Clone
Git: Fetch
Git: Pull
Git: Push
Git: Create Branch
Git: Checkout to
```

Beginner explanation:

- VS Code can do many Git tasks for you, but the terminal commands are still important because they work everywhere
- if you are unsure what VS Code is about to stage or commit, check the diff first

## Common Mistakes To Avoid

- committing secrets, kubeconfigs, or pull secrets
- pushing changes directly to protected branches
- mixing unrelated cluster updates in one commit
- using `git push --force` on shared branches without approval
- trusting generated YAML without reviewing the diff

Why this matters for beginners:

- Git mistakes are usually recoverable, but cluster configuration mistakes can affect other engineers and environments
- small, focused commits are easier to review, test, and roll back

## Troubleshooting

### Push Rejected

```bash
git fetch origin
git pull --ff-only
git push origin <branch>
```

If the branch is protected, open a pull request instead of retrying direct push.

### Wrong Author Information

```bash
git config --global user.name "<your-name>"
git config --global user.email "<your-email>"
```

This controls the name and email attached to future commits on your workstation.

### Authentication Fails

Check:

- remote URL protocol
- PAT or SSH key validity
- GitHub SSO authorization status
- credential helper state

### Diff Is Too Large To Review Safely

Break the work into smaller commits by component, cluster, or runbook.

## Beginner Safe Routine

When in doubt, use this short routine:

```bash
git status
git branch --show-current
git diff
git add <file>
git diff --staged
git commit -m "Describe the change clearly"
git push origin <branch>
```

This routine helps you slow down and check your work before you publish it.

## Quick Reference

```bash
git status
git switch -c docs/update-runbooks
git add README.md storage-configuration.md
git commit -m "Improve storage runbook and README"
git push origin docs/update-runbooks
```
