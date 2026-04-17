# OpenShift Learning Module 3: Automation with Ansible

This module helps you learn how to design repeatable OpenShift automation workflows with Ansible.

## Module Goals

By the end of this module, you should be able to:

- explain when to automate and when to run manual checks
- model cluster input data in clear, reusable files
- run template-driven jobs safely in a lab environment
- troubleshoot failed automation runs using structured checks

## Core Concepts

- idempotency means repeated runs should produce stable results
- inventory modeling should keep host and network data predictable
- separation of concerns keeps inputs, playbooks, and roles organized independently
- validation-first workflows fail early on bad values instead of failing late in deployment

## Recommended Practice Structure

```text
openshift-learning-automation/
├── inventories/
│   └── lab-01/
│       ├── hosts.yml
│       ├── network.yml
│       └── values.yml
├── playbooks/
│   ├── validate-inputs.yml
│   ├── generate-config.yml
│   └── run-install.yml
├── roles/
│   ├── validation/
│   ├── networking/
│   └── installation/
└── docs/
    ├── workflow.md
    └── troubleshooting.md
```

## Lab 1: Build A Minimal Inventory

Create a simple `hosts.yml` with control plane and worker groups.

```yaml
all:
  children:
    control_plane:
      hosts:
        cp-01.lab.example:
          ansible_host: 54.25.25.21
        cp-02.lab.example:
          ansible_host: 54.25.25.22
        cp-03.lab.example:
          ansible_host: 54.25.25.23
    workers:
      hosts:
        wk-01.lab.example:
          ansible_host: 54.25.25.31
        wk-02.lab.example:
          ansible_host: 54.25.25.32
```

## Lab 2: Model Network Inputs

Create `network.yml` with required fields:

```yaml
cluster_name: ocp-lab-01
default_gateway: 54.25.25.1
api_vip: 54.25.25.10
ingress_vip: 54.25.25.11
machine_network_cidr: 54.25.25.0/24
vlan_id: 218
```

## Lab 3: Add Validation Logic

Write a simple playbook that validates required inputs before any install logic runs.

```yaml
---
- name: Validate OpenShift automation inputs
  hosts: localhost
  gather_facts: false
  vars_files:
    - inventories/lab-01/network.yml
  tasks:
    - name: Check required network values
      ansible.builtin.assert:
        that:
          - cluster_name | length > 0
          - machine_network_cidr is match('^[0-9.]+/[0-9]+$')
          - api_vip != ingress_vip
        fail_msg: Network input validation failed.
```

## Lab 4: Review Execution Output

When practicing a run, review:

- failed tasks
- rendered variables
- generated config files
- whether the workflow stopped at the correct validation point

## Troubleshooting Routine

If automation fails unexpectedly:

1. verify inventory structure and indentation
2. verify network values and CIDR format
3. inspect the first failed task, not only the final summary
4. check whether validation failed early as designed
5. confirm the inputs match the intended lab scenario

## Weekly Checkpoint Questions

- Can I explain why idempotency matters in platform automation?
- Can I separate reusable logic from environment-specific inputs?
- Can I debug a failed run from the first useful error?

## Completion Criteria

You complete Module 3 when you can:

- build a clean inventory and network input set
- validate inputs before execution
- explain the flow of a simple automation run
- troubleshoot one broken run without guesswork