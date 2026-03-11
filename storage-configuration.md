
# Configuring Storage for an OpenShift Cluster Using GitOps Automation

## Overview
This guide explains how to configure external storage networking for a new OpenShift cluster using a GitOps automation workflow.

The process uses:
- Git repository configuration
- YAML configuration files
- Bastion host execution
- Label generation automation
- GitOps synchronization
- Network configuration validation

---

## Prerequisites

### Required Access
- Access to the automation Git repository
- SSH access to a Bastion server
- Access to the GitOps deployment system (such as ArgoCD)

### Required Information
You must know:
- Target cluster name
- Bare metal node hostnames
- Storage network VLAN IDs
- Storage network IP addresses
- Storage credentials stored in a secret manager

---

## Step 1 — Navigate to the Automation Git Repository

Open a browser and navigate to the Git repository managing cluster automation.

Example:

```
https://git.example.com/platform/cluster-automation
```

---

## Step 2 — Switch to the Initialization Branch

Change the repository branch to the cluster initialization branch.

Example:

```
cluster-init
```

---

## Step 3 — Locate Storage Configuration Files

Navigate to the storage policy templates directory.

Example:

```
policies/storage/files
```

---

## Step 4 — Connect to the Bastion Server

Open a terminal and connect to the Bastion host.

```
ssh username@bastion-server
```

---

## Step 5 — Navigate to the Cluster Networking Directory

```
cd storage-cluster-networking
```

---

## Step 6 — Copy an Existing Cluster YAML Template

Create a new YAML configuration file from an existing template.

```
cp example-cluster.yaml new-cluster.yaml
```

---

## Step 7 — Update the Cluster YAML Configuration

### Update Node Hostnames

```yaml
hostnames:
  - node01
  - node02
  - node03
```

---

### Configure Storage VLANs and Storage IPs

```yaml
vlans:
  - id: 100
    ips:
      - 10.10.10.54
      - 10.10.10.55
      - 10.10.10.56
```

---

### Configure Additional Storage VLAN

```yaml
vlans:
  - id: 200
    ips:
      - 10.20.20.54
      - 10.20.20.55
      - 10.20.20.56
```

---

## Step 8 — Configure Network MTU and Interfaces

```yaml
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

Always verify interface names match the actual node interfaces.

---

## Step 9 — Generate Node Labels Using the Automation Script

Run the label generation script.

```
sh generate-labels.sh new-cluster.yaml
```

This generates a labels file such as:

```
new-cluster-network-labels.yaml
```

---

## Step 10 — Edit the Automation Values File

Open the configuration file:

```
values.yaml
```

Paste the generated labels into the appropriate section.

---

## Step 11 — Configure Storage Operator Settings

```yaml
storage:
  enabled: true
  operator-name: storage-operator
  install-mode: automatic
  operator-source: certified-operators
  operator-channel: stable
```

---

## Step 12 — Configure Storage Backend Credentials

Retrieve storage credentials from the secret management system.

Example:

```
storage-backend-secret
```

Add it to the configuration:

```yaml
storage-credentials-secret: storage-backend-secret
```

---

## Step 13 — Sync Applications in the GitOps System

Open the GitOps dashboard and trigger synchronization.

Example:

```
Sync All Applications
```

Monitor until applications report:

```
Healthy
Synced
```

---

## Step 14 — Validate Network Configuration

Log into the cluster console and verify network policies.

Navigation example:

```
Networking → Network Policies
```

Verify:
- Policies exist for each worker node
- VLAN configuration matches YAML
- MTU values are applied correctly

---

## Expected Result

After completion:
- Storage networking is configured
- Cluster nodes have storage connectivity
- Network policies are applied automatically
- GitOps manages future configuration updates

---

## Beginner Tips

Always validate:

1. Network interface names  
2. VLAN IDs  
3. Storage IP addresses  
4. Cluster node hostnames  
5. Secret names  

Incorrect values here are the most common cause of deployment failures.

---

## Simplified Workflow

```
Edit YAML
   ↓
Run Automation Script
   ↓
Generate Labels
   ↓
Update Git Repository
   ↓
Sync GitOps Platform
   ↓
Cluster Applies Configuration
```
