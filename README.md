# OpenShift Platform Runbooks

This repository contains sanitized operational runbooks and engineer-facing cheatsheets for common OpenShift platform workflows.

The content is optimized for repeatable execution, safer documentation sharing, and faster onboarding. Every example uses placeholder values, private address space, or obviously non-production hostnames so the repository can be reused without leaking credentials or environment-specific details.

## Repository Goals

- standardize common OpenShift platform procedures
- document GitOps, storage, automation, and access workflows
- provide validated command and YAML examples
- reduce copy-paste risk by using sanitized placeholders
- give engineers a consistent place to start troubleshooting

## Document Index

- [Ansible Automation Platform runbook](./ansible-automation-platform.md)
- [OpenShift GitOps cluster platform runbook](./openshift-gitops-cluster-platform.md)
- [Storage configuration runbook](./storage-configuration.md)
- [Red Hat training access runbook](./redhat-training-access-runbook.md)
- [VS Code and GitHub setup runbook](./vscode-github-runbook.md)
- [Git command cheatsheet for OpenShift and VS Code](./git-command-cheatsheet.md)
- [OpenShift YAML cheatsheet](./openshift-yaml-cheatsheet.md)

## Documentation Standards Used In This Repo

All runbooks follow the same structure:

1. Purpose and scope
2. Prerequisites
3. Sanitized examples
4. Validation steps
5. Best practices
6. Troubleshooting

The examples in this repository use these conventions:

- domains such as `lab.example.com` and `apps.lab.example.com`
- private IP ranges such as `10.42.18.0/24`
- placeholders such as `<github-token>`, `<cluster-name>`, and `<redacted>`
- clearly fake serial numbers, hostnames, and usernames

## How To Use These Runbooks

1. Read the prerequisites section first.
2. Replace placeholders with environment-approved values.
3. Validate changes in a non-production environment before promotion.
4. Record approvals, ticket numbers, and change references outside this repo if required by your operating model.

## Playbook And Example Sanitization Guidance

This repository does not store production automation playbooks. It stores documentation and playbook-like examples. Those examples have been sanitized using these rules:

- no real tokens, pull secrets, or passwords
- no production cluster names or routable IP space
- no real employee identifiers or account names
- no destructive commands without context or safeguards
- no base64 blobs that appear to be live secrets

If you add new examples later, keep them sanitized to the same standard.

## Recommended Validation Tooling

These tools improve quality before you operationalize any YAML or automation derived from the docs:

- `yamllint`
- `ansible-lint`
- `helm template`
- `oc apply --dry-run=client -f <file>`
- `oc diff -f <file>`

## Suggested Workflow For Updates

1. Update the runbook or cheatsheet.
2. Verify commands and YAML examples still make sense.
3. Confirm placeholders are sanitized.
4. Add a troubleshooting note if you hit a new failure mode.
5. Keep the README index current.

## Audience

This repository is intended for:

- OpenShift platform engineers
- automation engineers
- SREs operating GitOps-managed clusters
- engineers onboarding to Red Hat and OpenShift workflows

## Contribution Expectations

When updating this repo:

- prefer precise steps over broad advice
- keep examples environment-neutral
- explain why a command is run when it is not obvious
- include validation checks after any risky step
- add troubleshooting entries for common operator mistakes

## Quick Start

If you are new to this repository, read in this order:

1. [README.md](./README.md)
2. [vscode-github-runbook.md](./vscode-github-runbook.md)
3. [git-command-cheatsheet.md](./git-command-cheatsheet.md)
4. The workflow-specific runbook you need for your task

## Maintenance Notes

- Review placeholders and versions regularly.
- Refresh OpenShift examples when cluster APIs or workflows change.
- Keep troubleshooting sections based on real incidents, not generic filler.
