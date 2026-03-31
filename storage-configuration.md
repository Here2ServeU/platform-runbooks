# OpenShift Learning Module 2: Storage and Network Configuration

This module focuses on the storage-network side of OpenShift preparation and validation.

## Module Goals

By the end of this module, you should be able to:

- model storage VLAN and IP plans in YAML
- map nodes to storage network requirements
- reason about MTU and interface choices
- validate configuration before sync or apply actions

## Concepts To Learn First

- VLAN segmentation for storage traffic
- node-level interface mapping
- MTU planning and consistency
- declarative network policy from versioned config

## Learning Flow

1. Gather cluster and node facts.
2. Define storage VLAN plans.
3. Assign storage IPs per node.
4. Define interfaces and MTU.
5. Generate labels or policies.
6. Review and validate config.
7. Apply and verify in cluster.

## Lab 1: Create A Cluster Storage Blueprint

Draft a YAML file with:

- node hostnames
- storage VLAN IDs
- storage IP mappings
- interface names
- MTU values

Example:

```yaml
hostnames:
  - node01
  - node02
  - node03

vlans:
  - id: 100
    ips:
      - 54.25.25.54
      - 54.25.25.55
      - 54.25.25.56
  - id: 200
    ips:
      - 54.25.25.154
      - 54.25.25.155
      - 54.25.25.156

mtu:
  value: 9000

interfaces:
  - name: eno1
    type: ethernet
  - name: eno2
    type: ethernet
  - name: bond0
    type: bond
```

## Lab 2: Validate Assumptions

Create a checklist and verify:

1. each node has expected interfaces
2. each VLAN has one IP per node
3. no duplicated IP addresses
4. MTU value is supported end-to-end
5. naming is consistent across files

## Lab 3: Simulate Label Or Policy Generation

Practice this workflow:

```text
Edit YAML -> Generate labels -> Insert labels into values -> Sync -> Validate status
```

Write down what artifact each step should produce.

## Lab 4: Post-Apply Validation Plan

After apply or sync, verify:

- expected policies exist
- node-level configuration matches intended VLAN and MTU values
- resources report healthy and synced status

## Troubleshooting Routine

If configuration does not apply as expected:

1. verify interface names on actual nodes
2. verify VLAN IDs and network reachability
3. verify generated labels were inserted in the correct section
4. verify sync status and failing resource details
5. verify referenced secret names exist

## Weekly Checkpoint Questions

- Can I explain why storage traffic needs its own VLAN strategy?
- Can I validate a network YAML without relying on guesswork?
- Can I identify whether an issue is data, generation, or apply related?

## Stretch Practice

- Build two storage profiles and compare them.
- Add a pre-merge checklist for storage network changes.
- Document one incident-style simulation and your root cause analysis.

## Completion Criteria

You complete Module 2 when you can:

- produce a clean storage network YAML from scratch
- validate it with a repeatable checklist
- explain post-apply verification steps clearly
- troubleshoot one broken scenario end-to-end