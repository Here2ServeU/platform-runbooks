# Ansible Automation Platform Runbook

## Purpose

Use this runbook to launch an OpenShift cluster installation workflow from Ansible Automation Platform (AAP) using sanitized inputs and operational best practices.

## Scope

This runbook covers:

- preparing required inputs before launch
- submitting an OpenShift cluster installation template in AAP
- reviewing generated inventory and configuration artifacts
- validating the workflow output before handoff or promotion

It does not replace environment-specific change approval, DNS reservation, IPAM, or platform architecture review.

## Prerequisites

- access to the AAP web console
- access to the approved automation template
- approved cluster name, base domain, VLAN, gateway, and VIP assignments
- approved credentials stored in AAP credential objects
- access to the destination Git repository if the workflow publishes artifacts

## Sanitized Example Inputs

Use placeholders while documenting or dry-running the process.

| Field | Sanitized example |
|---|---|
| Cluster name | `ocp-lab-cluster-01` |
| Base domain | `lab.example.com` |
| API VIP | `10.42.18.10` |
| Ingress VIP | `10.42.18.11` |
| Default gateway | `10.42.18.1` |
| Machine network CIDR | `10.42.18.0/24` |
| VLAN ID | `218` |
| Git branch | `feature/ocp-lab-cluster-01` |

### Sample Node Table

| Role | Hostname | Serial | IP address |
|---|---|---|---|
| Control plane | `ocp-cp-01.lab.example.com` | `SN-A91K2P7` | `10.42.18.21` |
| Control plane | `ocp-cp-02.lab.example.com` | `SN-B73M8Q4` | `10.42.18.22` |
| Control plane | `ocp-cp-03.lab.example.com` | `SN-C55R1T9` | `10.42.18.23` |
| Worker | `ocp-wk-01.lab.example.com` | `SN-D20L6X8` | `10.42.18.31` |
| Worker | `ocp-wk-02.lab.example.com` | `SN-E84N3Z1` | `10.42.18.32` |

## High-Level Workflow

1. Log in to AAP.
2. Open the approved OpenShift cluster install template.
3. Select the correct credential objects.
4. Enter cluster, network, and node details.
5. Review all generated values before launch.
6. Launch the job.
7. Validate generated files, logs, and downstream automation targets.

## Procedure

### 1. Open AAP

Browse to the AAP URL for your environment.

```text
https://aap.example.internal
```

### 2. Authenticate

Use your approved enterprise account or platform credential flow. Do not document live passwords in tickets, screenshots, or notes.

### 3. Open The Template

Navigate to:

```text
Automation Execution -> Templates
```

Find the approved template, for example:

```text
OpenShift Cluster Install
```

### 4. Collect Required Values Before Editing The Form

Prepare the following inputs before launching the template:

- cluster name
- base domain
- machine network CIDR
- API VIP
- ingress VIP
- gateway
- VLAN ID
- control plane and worker hostnames
- node IP addresses
- virtualization or hardware credentials
- Git repository destination if artifacts are published

### 5. Select Credential Objects

Typical AAP credentials for this workflow may include:

- virtualization platform credentials
- OpenShift service account credentials
- GitHub or Git server credentials
- out-of-band management credentials

Select credential objects by approved name. Do not create duplicate secrets unless the workflow owner requires it.

### 6. Enter Sanitized Or Approved Values

Example network values:

```text
Cluster Name: ocp-lab-cluster-01
Base Domain: lab.example.com
Default Gateway: 10.42.18.1
API Virtual IP: 10.42.18.10
Ingress Virtual IP: 10.42.18.11
VLAN ID: 218
Machine Network CIDR: 10.42.18.0/24
```

### 7. Review Generated Automation Inputs

Many AAP workflows generate inventory or value files before calling downstream playbooks or APIs. Expect outputs similar to the following.

#### Example `hosts.yml`

```yaml
all:
  children:
    control_plane:
      hosts:
        ocp-cp-01.lab.example.com:
          ansible_host: 10.42.18.21
        ocp-cp-02.lab.example.com:
          ansible_host: 10.42.18.22
        ocp-cp-03.lab.example.com:
          ansible_host: 10.42.18.23
    workers:
      hosts:
        ocp-wk-01.lab.example.com:
          ansible_host: 10.42.18.31
        ocp-wk-02.lab.example.com:
          ansible_host: 10.42.18.32
```

#### Example `network.yml`

```yaml
cluster_name: ocp-lab-cluster-01
base_domain: lab.example.com
default_gateway: 10.42.18.1
api_vip: 10.42.18.10
ingress_vip: 10.42.18.11
vlan_id: 218
machine_network_cidr: 10.42.18.0/24
```

### 8. Launch The Job

Use the template review page to verify the final values, then launch the job.

### 9. Monitor Execution

Navigate to:

```text
Automation Execution -> Jobs
```

Check for:

- job status
- failed tasks
- rendered variable values
- downstream repository or API update actions

## Sanitized Playbook Example

This repository does not include live playbooks, but the example below shows the minimum standard for documenting them safely.

```yaml
---
- name: Validate OpenShift cluster input values
  hosts: localhost
  gather_facts: false
  vars:
    cluster_name: ocp-lab-cluster-01
    machine_network_cidr: 10.42.18.0/24
    api_vip: 10.42.18.10
    ingress_vip: 10.42.18.11
  tasks:
    - name: Ensure cluster name is provided
      ansible.builtin.assert:
        that:
          - cluster_name | length > 0
          - machine_network_cidr is match('^[0-9.]+/[0-9]+$')
          - api_vip != ingress_vip
        fail_msg: Input validation failed. Review cluster name, CIDR, and VIP assignments.
```

## Validation Checklist

Validate these items before marking the job successful:

- the AAP job completed without failed tasks
- the cluster name and base domain match the approved request
- generated inventory and network files match the submitted values
- no placeholder secrets or test credentials leaked into Git
- downstream Git commits or pull requests target the correct branch
- workflow logs clearly show the target environment and template version

## Best Practices

- store credentials only in AAP credential objects or the approved secret backend
- validate IPs, hostnames, and VLANs before launch
- use branch names that map to a single cluster request
- lint generated Ansible and YAML before production promotion
- keep generated config separate from reusable roles and templates
- make the job template idempotent where possible
- document approval, change ID, and rollback owner outside the runbook if required

## Troubleshooting

### Template Does Not Appear

- confirm your AAP organization and permissions
- verify the template is not hidden or deprecated
- check whether the template moved to a different execution environment or team folder

### Credential Selection Fails

- confirm the credential object is shared with your team
- verify the credential type matches the expected input field
- check whether the underlying secret or token has expired

### Job Fails During Input Validation

- compare the submitted CIDR, VIP, and gateway values against the approved network design
- confirm there are no duplicate node IPs or hostnames
- review whether the cluster name violates local naming policy

### Job Updates The Wrong Git Branch

- verify the template survey or extra vars contain the expected branch name
- review default values in the job template
- check whether a webhook or downstream script overrides the target branch

### Generated Files Look Incorrect

- compare the rendered inventory with the request form input
- inspect variable precedence if defaults override survey answers
- review any Jinja templating logic for missing conditionals or bad defaults

### Escalation Data To Capture

When escalating a failed run, include:

- AAP job ID
- template name and version
- time of execution
- sanitized copy of the submitted values
- first failing task name
- repository or environment affected
