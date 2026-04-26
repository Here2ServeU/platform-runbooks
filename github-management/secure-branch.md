# GitHub — Secure Main Branch Runbook

**Repo:** `Any`  
**Author:** Emmanuel Naweji  
**Last Updated:** April 2026

---

## What This Does

Securing the main branch means no one — not even you — can push code directly to `main` without it going through a pull request and passing checks first. This protects the live codebase from accidental overwrites or bad code going straight to production.

---

## Step 1 — Go to Branch Protection Settings

1. Open your repo on GitHub
2. Click **Settings** (top menu of the repo)
3. Click **Branches** in the left sidebar
4. Under **Branch protection rules**, click **Add rule**

---

## Step 2 — Set the Branch Name Pattern

In the **Branch name pattern** field type exactly:

```
main
```

---

## Step 3 — Enable These Settings

Check each of the following boxes:

### Require a pull request before merging
- Set **Required approvals** to `1`
- At least one other person must review and approve before anything merges

### Require status checks to pass before merging
- Check **Require branches to be up to date before merging**
- Add any CI checks you have running (GitHub Actions, tests, etc.)

### Require conversation resolution before merging
- All review comments must be resolved before the PR can merge

### Do not allow bypassing the above settings
- This applies the rules to admins too — including you
- Critical for a clean repo

### Restrict who can push to matching branches
- Add only the people who should ever touch main directly
- In most cases, no one — everything goes through PRs

---

## Step 4 — Optional But Recommended

### Require signed commits
Commits must be cryptographically signed. Adds a verified badge and confirms identity.

### Require linear history
Prevents merge commits. Forces squash or rebase merges only — keeps the commit history clean and readable.

### Lock branch
Only turn this on if you want main to be completely read-only. Useful for archived or dormant repos — not for active development.

---

## Step 5 — Save the Rule

Click **Create** at the bottom of the page.

---

## Step 6 — Verify It's Working

Try to push directly to main from your terminal:

```bash
git checkout main
git push origin main
```

You should get an error like:

```
remote: error: GH006: Protected branch update failed
```

That means it is working correctly.

Then create a test branch, open a PR, and verify the merge button is blocked until approval conditions are met.

---

## The Correct Workflow After This

Every change to the codebase now follows this flow:

```
Create a branch -> Write code -> Push branch -> Open Pull Request -> Get reviewed -> Merge to main
```

### Branch Naming Convention

| Type | Format | Example |
|------|--------|---------|
| New feature | `feature/description` | `feature/career-advisor-module` |
| Bug fix | `fix/description` | `fix/login-redirect` |
| Content update | `content/description` | `content/update-pricing` |
| Emergency fix | `hotfix/description` | `hotfix/broken-exam-chat` |

---

## Team Roles

Go to **Settings -> Collaborators and teams** and assign:

| Person | Role | Can Merge? |
|--------|------|------------|
| Emmanuel | Admin | Yes (via PR) |

---

## Emergency Override (If Absolutely Needed)

If you need to bypass the rules in a true emergency:

1. Go to **Settings -> Branches -> Edit rule**
2. Temporarily uncheck **Do not allow bypassing**
3. Push your fix
4. Re-enable the setting immediately after

> NOTE: Never leave bypass mode on.

---

## Quick Reference

| What you want to do | How |
|---------------------|-----|
| Push code | Create a branch, open a PR |
| Merge to main | Get at least 1 approval, then merge |
| See who has access | Settings -> Collaborators |
| Edit protection rules | Settings -> Branches -> Edit |
| Disable a rule temporarily | Edit rule -> uncheck -> save -> re-enable after |

---

> Set this up before Nehemie starts pushing code to the repo. Takes 5 minutes and protects the entire production codebase.

---

*Emmanuel Naweji · 2026 · info@transformed2succeed.com*
