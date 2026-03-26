# OpenShift GitOps Cluster Platform Runbook

## Purpose

Use this runbook to deploy or update an OpenShift cluster installation workflow managed through GitOps, Helm, and Advanced Cluster Management (ACM).

## Scope

This runbook covers:

- preparing cluster-specific secrets
- connecting Git repositories to Argo CD
- updating Helm values in a controlled way
- rendering and validating manifests before deployment
- deploying the bootstrap chart and observing cluster creation

## Prerequisites

- access to the hub cluster OpenShift console
- access to OpenShift GitOps and Argo CD
- access to ACM or the management cluster
- access to the Git repository used for cluster definitions
- access to a bastion host with `git`, `oc`, and `helm`
- approved cluster networking, host inventory, BMC, and pull-secret data


## High-Level Flow

```text
Create secrets
-> connect Git repository to Argo CD
-> prepare Helm values
-> render templates locally
-> commit reviewed changes
-> deploy bootstrap chart
-> watch Argo CD sync
-> monitor ACM resources
-> validate cluster installation
```

## Procedure

### 1. Prepare The Hub Cluster Namespace And Secrets

Create or verify the target namespace:

```text
cluster-your-secrets
```

Required secrets usually include:

- pull secret for image registry access
- BMC or out-of-band management credential secret
- optional CA trust bundle secret if your environment requires it

Use sanitized placeholders in documentation. Do not store real pull secrets or passwords in Git.

#### Secret Example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bmc-credential
  namespace: cluster-your-secrets
type: Opaque
stringData:
  username: <redacted-username>
  password: <redacted-password>
```

### 2. Create Or Review The Argo CD Project

In Argo CD, create or validate the project that permits the repository and destination cluster.

Confirm:

- source repository URL is correct
- destination targets the hub or management cluster as intended
- cluster-scoped resources required by ACM are permitted

### 3. Connect The Git Repository

In Argo CD:

```text
Settings -> Repositories
```

Add the repository using the approved authentication method.

### 4. Prepare A Bastion Workspace

```bash
mkdir -p ~/git
cd ~/git
git clone <repository-url>
cd <repository-folder>
```

Switch to the correct branch:

```bash
git checkout new-site-branch
git pull --ff-only
```

### 5. Update Host Inventory Values

Example `allHosts.yaml`:

```yaml
hosts:
  - name: master-01
    ip: 10.42.18.21
    mac: "AA:BB:CC:11:22:33"
    vlan: 218
    bmcIP: 10.42.17.21
    bmcEndpoint: redfish/v1/Systems/System.Embedded.1

  - name: master-02
    ip: 10.42.18.22
    mac: "AA:BB:CC:11:22:34"
    vlan: 218
    bmcIP: 10.42.17.22
    bmcEndpoint: redfish/v1/Systems/System.Embedded.1
```

### 6. Review Global Values

Example `globalValues.yaml`:

```yaml
clusterSet: production-east
secretSourceNamespace: cluster-your-secrets
recreateHosts: true
caTrustBundle: ""
```

Validate that global values match the cluster set, namespace, and trust requirements for your environment.

### 7. Create Cluster-Specific Values

Example `ocp-lab-01.yaml`:

```yaml
clusterName: ocp-lab-01
baseDomain: lab.example.com
clusterImageSet: img4.16.10-x86-64
bmcCredentialSecret: bmc-credential
pullSecretRef: cluster-install-pull-secret
sshPublicKey: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCsanitizedExample
networking:
  machineNetworkCIDR: 10.42.18.0/24
  apiVIP: 10.42.18.10
  ingressVIP: 10.42.18.11
  gateway: 10.42.18.1
  vlan: 218
```

### 8. Review Bootstrap Chart Values

Example bootstrap chart setting:

```yaml
targetRevision: new-site-branch
```

If this points to the wrong branch, Argo CD can reconcile the wrong cluster definition.

### 9. Render Helm Templates Before Commit

```bash
helm template ./helm-charts/cluster-install \
  -f ./helm-charts/cluster-install/allHosts.yaml \
  -f ./helm-charts/cluster-install/globalValues.yaml \
  -f ./helm-charts/cluster-install/ocp-lab-01.yaml
```

Review the output for:

- correct namespaces
- correct secret references
- correct cluster name and domain
- expected ACM resource kinds

### 10. Review, Commit, And Push

```bash
git status
git add helm-charts/cluster-install
git commit -m "Add sanitized cluster install config for ocp-lab-01"
git push
```

### 11. Authenticate To The Hub Cluster

Use the OpenShift console login command or your approved enterprise login flow.

```bash
oc login --server=https://api.hub.lab.example.com:6443 --token=<redacted-token>
```

### 12. Deploy The Bootstrap Chart

```bash
helm install ocp-lab-01-bootstrap ./helm-charts/cluster-install-bootstrap
```

### 13. Monitor Argo CD And ACM

Watch for:

- Argo CD application created
- sync status moves to `Synced`
- health status moves to `Healthy`
- ACM resources such as `ManagedCluster`, `ClusterDeployment`, and `AgentClusterInstall` appear

## Validation Checklist

Run these checks after deployment:

```bash
oc get applications.argoproj.io -A
oc get managedcluster
oc get clusterdeployment -A
oc get agentclusterinstall -A
oc get baremetalhost -A
```

Success indicators:

- Argo CD shows the application as synced and healthy
- ACM resources are created in the correct namespace
- no secret references are missing
- host records match expected MAC, IP, and BMC data

## Best Practices

- use `stringData` in examples rather than base64-encoded live-looking secret values
- render Helm locally before pushing changes
- change one cluster definition per branch or commit when possible
- keep bootstrap chart branch references explicit
- verify every secret name from the consuming manifest, not from memory
- validate resource diffs before syncing to shared environments
- separate lab validation from production promotion

## Troubleshooting

### Repository Authentication Fails In Argo CD

- verify the repository URL and protocol
- confirm the token or SSH key is valid and scoped correctly
- check whether your Git provider requires SSO authorization

### Helm Rendering Fails

- confirm the chart path is correct
- verify all required values files are passed in the correct order
- check for missing keys referenced by templates

### Argo CD Application Is Out Of Sync Or Degraded

- inspect the application events and diff view
- confirm `targetRevision` points to the expected branch
- verify referenced secrets and namespaces already exist

### `oc get nodes` Or Cluster Queries Fail

- confirm your `oc login` target is the hub cluster, not the future spoke cluster
- verify your token is not expired
- confirm the bastion host has network access to the API endpoint

### Cluster Does Not Start Provisioning

- check the BMC secret values and endpoint path
- validate host MAC addresses and network assignments
- verify the pull secret is present and correctly referenced
- confirm the cluster image set exists in the management environment

### Resources Are Created In The Wrong Namespace

- review namespace values in the Helm chart
- inspect any global default values or chart helper templates
- verify Argo CD destination and namespace overrides

### Escalation Data To Capture

Capture the following before escalation:

- Git commit SHA
- Argo CD application name
- cluster namespace
- first failing ACM resource
- exact Helm command used
- sanitized values file names involved
