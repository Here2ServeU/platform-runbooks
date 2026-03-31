# Personal OpenShift Learning Roadmap

This repository is a personal OpenShift learning roadmap built around practical study modules, repeatable labs, and lightweight reference material.

The goal is to build working knowledge across GitOps, storage and networking, and automation without mixing in client-specific context.

## Roadmap Structure

1. Module 1: GitOps Foundations and Cluster Lifecycle
   - Guide: `openshift-gitops-cluster-platform.md`
   - Focus: Git, Argo CD, Helm, sync flow, and reconciliation behavior

2. Module 2: Storage and Network Configuration
   - Guide: `storage-configuration.md`
   - Focus: VLAN planning, IP allocation, MTU decisions, and validation workflow

3. Module 3: Automation with Ansible
   - Guide: `ansible-automation-platform.md`
   - Focus: inventory modeling, input validation, repeatable automation, and troubleshooting

## Suggested Study Cadence

- Week 1: Complete Module 1 and write your own GitOps glossary.
- Week 2: Complete Module 2 and build one clean storage-network YAML example.
- Week 3: Complete Module 3 and simulate one end-to-end automation workflow.
- Week 4: Review all modules and create a short capstone checklist for yourself.

## How To Use This Repository

- Work through the modules in order.
- Keep personal notes for terms, failure patterns, and fixes.
- Use a lab or sandbox environment when testing commands.
- Repeat labs until you can explain the workflow without reading every step.

## Shared Example Conventions

To keep examples consistent across the repository, these docs use:

- the sample network `54.25.25.0/24`
- hostnames under `lab.example.com`
- placeholder secrets such as `<redacted>`
- YAML and command examples intended for learning, not production copy-paste

## Outcome Goals

By the end of this roadmap, you should be able to:

- explain how OpenShift GitOps workflows operate
- model cluster and network inputs clearly
- validate configuration before apply or sync steps
- structure Ansible inputs for repeatable automation
- troubleshoot common OpenShift configuration failures methodically

## Repository Notes

- `git-command-cheatsheet.md` provides quick Git references.
- `openshift-yaml-cheatsheet.md` helps with YAML structure and habits.
- the remaining markdown files can be used as supporting reference material alongside the core roadmap modules

## License

This repository is licensed under the MIT License. See `LICENSE`.