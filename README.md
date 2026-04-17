# Runbook Library

A personal library of runbooks and reference material for platform operations work. Organized by topic so each runbook can be used standalone.

## Categories

### OpenShift — [openshift/](openshift/)

Runbooks and references for OpenShift cluster lifecycle, GitOps, storage, networking, and automation.

- [install.md](openshift/install.md) — cluster install runbook
- [gitops-cluster-platform.md](openshift/gitops-cluster-platform.md) — Git, Argo CD, Helm, sync flow, reconciliation
- [storage-configuration.md](openshift/storage-configuration.md) — VLAN planning, IP allocation, MTU, validation
- [ansible-automation-platform.md](openshift/ansible-automation-platform.md) — inventory modeling, input validation, automation
- [yaml-cheatsheet.md](openshift/yaml-cheatsheet.md) — OpenShift YAML structure and habits
- [redhat-training-access.md](openshift/redhat-training-access.md) — Red Hat training environment access

### Certificates — [certificates/](certificates/)

Runbooks for certificate renewal, generation, and deployment.

- [capsule-certificate-renewal.md](certificates/capsule-certificate-renewal.md) — Satellite capsule certificate renewal and generation (manual, auto-renewal disabled)

### Reference — [reference/](reference/)

Supporting cheatsheets and workflow references used alongside the runbooks above.

- [git-command-cheatsheet.md](reference/git-command-cheatsheet.md) — Git command quick reference
- [vscode-github-runbook.md](reference/vscode-github-runbook.md) — VS Code and GitHub workflow

## How To Use This Library

- Each runbook is self-contained — start from the relevant file for your task.
- Runbooks use placeholder values (e.g. `<servername>`, `<redacted>`) — substitute your own before running commands.
- Shared example conventions across OpenShift docs:
  - sample network `54.25.25.0/24`
  - hostnames under `lab.example.com`
  - YAML and commands are illustrative, not production copy-paste

## Adding a New Runbook

1. Pick the matching category folder (`openshift/`, `certificates/`, `reference/`), or create a new one if none fits.
2. Add the runbook as a markdown file with a clear `Runbook:` title and numbered steps.
3. Link it from the matching section in this README.

## License

This repository is licensed under the MIT License. See [LICENSE](LICENSE).
