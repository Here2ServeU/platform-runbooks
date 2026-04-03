# Storage Configuration Runbook  
## Trident + NMState Label Automation

This guide explains how to generate and apply storage (Trident) and network (NMState) labels for a new OpenShift cluster using GitOps workflows.

---

# Overview

This process automates:

- NMState network configuration (VLANs, IPs)
- Trident storage configuration (backend, storage classes)
- Cluster-specific label generation
- Git-based deployment via pull requests

---

# Prerequisites

- Access to GitHub repository:
  atlas-lab-cluster-configs
- Access to DevSpaces (or local workspace)
- Basic Git knowledge
- Cluster name available
- Network details:
  - Hostnames
  - VLAN IDs
  - IP addresses
  - CIDR / prefix length

---

# Step 1 — Prepare Configuration Repository

1. Clone the repository:
   git clone https://github.com/example/atlas-lab-cluster-configs.git

2. Create a branch named after your cluster:
   feature/nebula-x-y-z

---

# Step 2 — Generate NMState Labels

## Navigate to scripts

cd storage-config/storage-scripts/nmstate-label-generator

## Copy example

cp example_lab.yaml nebula-x-y-z.yaml

## Update file

Edit and update:
- hostnames
- VLAN IDs
- IP addresses
- prefix length

## Generate labels

sh generate-labels.sh nebula-x-y-z.yaml

## Output

nebula-x-y-z.autoshift-nmstate-labels.yaml

---

# Step 3 — Update values file

Open:

values.cluster-hub.yaml

Add:

clusters:
  nebula-x-y-z:
    labels:

Paste generated labels under this section.

---

# Step 4 — Add Trident Labels

Add:

trident: 'true'
trident-name: trident-operator
trident-install-plan-approval: Automatic
trident-source: certified-operators
trident-source-namespace: openshift-marketplace
trident-channel: stable
trident-creds-secret: atlas-backend-secret
trident-secrets-namespace: cluster-install-secrets
trident-config-1: nebula-storage-sc
trident-config-2: nebula-backend-config
trident-config-3: trident-csi-snapclass
trident-vault-secret: 'false'

---

# Step 5 — Storage Files

cd autoshiftv2/policies/trident/files

cp template.sc.yaml nebula.sc.yaml
cp template.tbc.yaml nebula.tbc.yaml

Update backend config as needed.

---

# Step 6 — Commit and Push

git add .
git commit -m "Add config for nebula-x-y-z"
git push

---

# Step 7 — Create Pull Request

Go to:
software/autoshiftv2

Click "Compare & Pull Request"

Set:
- Base: main
- Compare: feature/nebula-x-y-z

Assign reviewers:
- John Doe

Submit PR.

---

# Final Outcome

- Network configured automatically
- Storage provisioned automatically
- Cluster ready for workloads

---

# Summary

- Generate labels
- Update values file
- Add storage config
- Commit and push
- Create PR
