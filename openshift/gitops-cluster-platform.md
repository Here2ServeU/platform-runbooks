# OpenShift Learning Module 1: GitOps Foundations and Cluster Lifecycle

This module is a personal learning guide for understanding how GitOps patterns support OpenShift cluster operations.

## Module Goals

By the end of this module, you should be able to:

- explain how Git, Argo CD, and Helm work together
- describe the lifecycle of a cluster configuration change
- render and review Helm templates before applying changes
- identify common sync and configuration issues

## Concepts To Learn First

- GitOps means desired state is stored in Git and reconciled continuously
- Argo CD watches Git and applies the declared state to the cluster
- Helm provides reusable templates and values-driven configuration
- a hub cluster acts as the management point in fleet-style workflows

## Learning Flow

1. Understand the architecture.
2. Create a minimal GitOps project.
3. Connect a repository.
4. Add simple Helm values.
5. Render templates locally.
6. Commit and sync.
7. Observe drift and reconciliation.

## Lab 1: Build A Mental Model

Sketch this flow in your notes:

```text
Git commit -> Argo CD detects change -> Sync starts -> Resources updated -> Cluster converges
```

Write one sentence for each arrow describing what can fail and how to validate it.

## Lab 2: Create A Starter Repository Structure

Use this structure as a practice baseline:

```text
openshift-learning-gitops/
├── README.md
├── helm/
│   ├── Chart.yaml
│   ├── templates/
│   └── values/
│       └── lab-cluster.yaml
├── docs/
│   ├── architecture.md
│   ├── glossary.md
│   └── troubleshooting.md
└── scripts/
    ├── render.sh
    └── verify.sh
```

## Lab 3: Practice Value Modeling

Create a cluster values file with placeholders for:

- cluster name
- base domain
- machine network CIDR
- API VIP
- ingress VIP
- gateway
- VLAN

Example learning template:

```yaml
clusterName: lab-ocp-01
baseDomain: lab.example.com
networking:
  machineNetworkCIDR: 54.25.25.0/24
  apiVIP: 54.25.25.10
  ingressVIP: 54.25.25.11
  gateway: 54.25.25.1
  vlan: 218
```

## Lab 4: Render Before Sync

Practice rendering a chart locally before any sync step:

```bash
helm template ./helm -f ./helm/values/lab-cluster.yaml
```

Check for:

- correct names and namespaces
- expected secret references
- intended network values
- obvious template or schema errors

## Lab 5: Validation Routine

After a sync, practice validating the cluster view with checks such as:

```bash
oc get applications.argoproj.io -A
oc get managedcluster
oc get clusterdeployment -A
```

Write down what success looks like for each command.

## Troubleshooting Routine

If the GitOps flow does not behave as expected:

1. verify the correct branch and values file were used
2. render templates locally again
3. inspect Argo CD sync and health state
4. inspect resource events for the failing object
5. compare desired state in Git with live cluster state

## Weekly Checkpoint Questions

- Can I explain what Argo CD reconciles and when?
- Can I tell the difference between a template problem and a sync problem?
- Can I validate a values change before applying it?

## Completion Criteria

You complete Module 1 when you can:

- explain the full GitOps change path clearly
- create a basic values file without guesswork
- render templates and catch mistakes before sync
- troubleshoot one broken sync scenario end-to-end