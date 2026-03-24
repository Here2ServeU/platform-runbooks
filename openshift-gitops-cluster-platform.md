# OpenShift Cluster Deployment Using Custom Helm Charts with GitOps
## Complete Beginner Step-by-Step Guide

**Author:** Emmanuel Naweji  
**Format:** Beginner-friendly GitHub documentation  
**Audience:** Complete beginners

---

## 1. What This Guide Covers

This README combines two workflows into one beginner-friendly guide:

1. Deploying an OpenShift cluster using custom Helm charts with GitOps
2. Creating CVI secrets and running the bootstrap deployment

It explains:

- what to prepare
- what secrets to create
- what files to update
- how Git, Argo CD, Helm, and ACM work together
- how to deploy and validate the cluster from a bastion host

---

## 2. What You Are Building

You are deploying an OpenShift cluster using:

- **Git** to store configuration files
- **GitOps** to automatically apply changes from Git
- **Argo CD** to watch the Git repository
- **Helm charts** to package the cluster configuration
- **ACM** to provision and manage the cluster
- **Secrets** to safely store credentials needed during installation

Think of it like this:

- **Git** = your notebook
- **Helm** = your packaging box
- **Argo CD** = your delivery robot
- **ACM** = the builder
- **Secret** = your locked credential box
- **OpenShift cluster** = the finished house

---

## 3. Big Picture

```text
Create Secrets
  -> Connect Git Repo
  -> Create Argo CD Project
  -> Prepare Helm Files
  -> Update CVI Branch
  -> Push Files to Git
  -> Deploy Bootstrap Helm Chart
  -> Argo CD Syncs
  -> ACM Provisions
  -> Cluster Becomes Managed
```

---

## 4. What You Need Before Starting

You should have access to these items:

- OpenShift hub cluster console
- Argo CD / GitOps in OpenShift
- ACM / hub cluster
- A Git repository
- A bastion host or terminal
- `oc` CLI
- `helm`
- Cluster install values
- Pull secret
- BMC credentials
- SSH public key
- Hostnames, IPs, MAC addresses, VLAN, gateway, and network details
- The target namespace:
  - `cluster-your-secrets`

---

## 5. Words You Should Know

### Hub Cluster
This is the main OpenShift cluster where you control everything.

### GitOps
This means OpenShift watches Git and applies changes automatically.

### Argo CD
This is the tool that reads Git and syncs resources to OpenShift.

### Helm Chart
This is a folder of templates used to build Kubernetes/OpenShift resources.

### ACM
ACM means Advanced Cluster Management. It helps create and manage OpenShift clusters.

### Bastion
This is a server or terminal you use to prepare files and run commands.

### Secret
A secret stores things like passwords, pull secrets, and credentials.

### BMC
BMC means Baseboard Management Controller. It is used to manage physical servers remotely using tools like iDRAC or iLO.

### Your Branch
This is the Git branch used for a specific site or cluster deployment.

---

## 6. Step 1 - Prepare the Hub Cluster

Log in to the **Hub cluster OpenShift console**.

Create this namespace:

```text
cluster-your-secrets
```

Create these required secrets:

1. **cluster-install-pull-secret**
   - used for registry authentication
   - type: image pull secret

2. **bmc-credential**
   - used for BMC, iDRAC, or iLO access
   - contains the username and password for bare metal management

Simple explanation:

- The **pull secret** helps the cluster download required images
- The **BMC secret** helps the platform talk to the physical machines

---

## 7. Step 2 - Use the Correct ID

If no VASI ID is available, use the alternate identifier.

### Action
- Use the **OCS ID** when VASI OCS is not available

Record the ID you are using before you continue.

---

## 8. Step 3 - Create or Copy the BMC Secret

Open the console and go to:

- **Administrator**
- **Workloads**
- **Secrets**

Open the target project:

```text
cluster-your-secrets
```

Find an existing secret that is close to what you need and use it as a template.

Example:

```text
company-bmc-credential
```

Copy its YAML and keep only these reusable fields:

```yaml
kind:
apiVersion:
metadata:
  name:
  namespace:
type:
data:
```

Remove generated fields such as:

- `creationTimestamp`
- `resourceVersion`
- `uid`
- `managedFields`
- `selfLink`

Update:

- `metadata.name`
- `metadata.namespace`
- `type`
- `data.username`
- `data.password`

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: company-bmc-credential
  namespace: cluster-your-secrets
type: Opaque
data:
  username: <base64-encoded-username>
  password: <base64-encoded-password>
```

Then use **Import YAML**, paste the secret, verify the fields, and click **Create**.

---

## 9. Step 4 - Create a GitOps Project

In OpenShift GitOps / Argo CD:

```text
Settings -> Projects -> Create Project
```

Configure:

- **Source repository**
- **Destination** = hub cluster
- **Resource allow list** = cluster and namespace resources

This tells Argo CD:

- which repo to trust
- where it is allowed to deploy
- what kinds of resources it can create

---

## 10. Step 5 - Connect the Git Repository to GitOps

Open Argo CD from the hub cluster OpenShift console:

1. Go to the top right of the console
2. Open the application menu
3. Select **Infra Argo CD**

Then go to:

```text
Settings -> Repositories
```

Add your repository using:

- repository URL
- username
- token or password

---

## 11. Step 6 - Prepare Your Working Folder on the Bastion

Create a folder:

```bash
mkdir git
cd git
```

If your environment uses Git credential manager, sign in using the method your team supports.

Example:

```bash
git credential-manager github login --url https://example.git.server
```

---

## 12. Step 7 - Clone the Cluster Install Repository

```bash
git clone <your-repository-url>
cd <your-repository-folder>
```

If your team wants you to work in a different branch:

```bash
git fetch origin source_branch:new_branch
git checkout new_branch
```

---

## 13. Step 8 - Go to the Correct CVI Branch

If your process uses a branch specific to a CVI site, switch to that branch.

```bash
git checkout <cvi-branch-name>
git pull
```

Check current branch:

```bash
git branch --show-current
```

---

## 14. Step 9 - Go to the Helm Chart Folder

```bash
cd helm-charts/cluster-install
```

This is where the templates and values live.

---

## 15. Step 10 - Update the Main Host File

Find:

```text
allHosts.yaml
```

Update it with:

- IP address
- MAC address
- VLAN
- BMC IP
- BMC endpoint
- hostname

Example:

```yaml
hosts:
  - name: master-01
    ip: 190.25.50.21
    mac: "AA:BB:CC:11:22:33"
    vlan: 510
    bmcIP: 190.25.40.21
    bmcEndpoint: "redfish/v1/Systems/System.Embedded.1"

  - name: master-02
    ip: 190.25.50.22
    mac: "AA:BB:CC:11:22:34"
    vlan: 210
    bmcIP: 190.25.40.22
    bmcEndpoint: "redfish/v1/Systems/System.Embedded.1"

  - name: worker-01
    ip: 10.20.30.31
    mac: "AA:BB:CC:11:22:41"
    vlan: 510
    bmcIP: 190.25.40.31
    bmcEndpoint: "redfish/v1/Systems/System.Embedded.1"
```

---

## 16. Step 11 - Review the Global Values File

Find:

```text
globalValues.yaml
```

Review or update:

- `clusterSet`
- `secretSourceNamespace`
- `recreateHosts`
- `caTrustBundle`

Example:

```yaml
clusterSet: production-east
secretSourceNamespace: cluster-your-secrets
recreateHosts: true
caTrustBundle: ""
```

---

## 17. Step 12 - Create the Cluster-Specific Values File

Create a file like:

```text
ocp-lab-01.yaml
```

Include:

- cluster name
- base domain
- cluster image set
- BMC credential reference
- pull secret reference
- networking values
- SSH public key

Example:

```yaml
clusterName: ocp-lab-01
baseDomain: lab.example.com
clusterImageSet: img4.16.10-x86-64
bmcCredentialSecret: bmc-credential
pullSecretRef: cluster-install-pull-secret
sshPublicKey: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCexamplekey"
networking:
  machineNetworkCIDR: 190.25.50.0/24
  apiVIP: 190.25.50.10
  ingressVIP: 190.25.50.11
  gateway: 190.25.50.1
  vlan: 510
```

---

## 18. Step 13 - Update Bootstrap Values

Locate:

```text
helm-charts/cluster-install-bootstrap/values.yaml
```

Find:

```yaml
targetRevision: ""
```

or:

```yaml
targetRevision: main
```

Update it to the correct branch for your CVI site.

Example:

```yaml
targetRevision: det-site-branch
```

---

## 19. Step 14 - Generate the Helm Template

```bash
helm template ./ -f allHosts.yaml -f globalValues.yaml -f ocp-lab-01.yaml
```

This combines host values, global values, and cluster-specific values into Kubernetes/OpenShift resources.

---

## 20. Step 15 - Review and Save Your Git Changes

```bash
git status
git add .
git commit -m "Add cluster install config for ocp-lab-01"
git push
```

---

## 21. Step 16 - Pull Latest Changes on the Bastion

```bash
ssh <user>@<bastion-host>
git pull
```

Use Git Bash on Windows if that is your normal shell.

---

## 22. Step 17 - Log In to OpenShift from ACM

In the hub console:

1. Click your username at the top right
2. Select **Copy login command**

Paste it into the bastion terminal.

Example:

```bash
oc login --token=sha256~exampletoken --server=https://api.hub.example.com:6443
```

---

## 23. Step 18 - Validate Access

```bash
oc get nodes
```

If the command returns the node list, your login is working.

---

## 24. Step 19 - Deploy the Helm Chart

Example:

```bash
helm install ocp-lab-01 cluster-install-bootstrap
```

Or:

```bash
helm install det-cluster-bootstrap ./helm-charts/cluster-install-bootstrap
```

This creates the GitOps application for the cluster install.

---

## 25. Step 20 - Watch the Argo CD Application

After deployment, open Argo CD and check the application.

You want to see:

- app created
- app synced
- app healthy

If it shows degraded or broken, read the error and debug the failing resource.

---

## 26. Step 21 - Monitor Installation Progress

### Option 1 - API Explorer

Go to:

```text
API Explorer -> ClusterInstance (under All Projects) -> Instances
```

Then:

1. Select your cluster
2. Open **Details**
3. Scroll down to conditions, status, and issue updates

### Option 2 - ACM / Fleet Management
Monitor the cluster install from ACM.

### Option 3 - Virtual Console
Monitor the machine boot sequence using the Out-of-Band IP or BMC console.

---

## 27. What Happens Next

After deployment:

1. Argo CD syncs resources
2. ACM detects a new `ManagedCluster`
3. The nodes boot from ISO or discovery image
4. The cluster install begins
5. The cluster auto-imports into its cluster set
6. The cluster becomes managed

---

## 28. Final Flow in Very Simple Words

```text
Create secrets
-> connect Git repo
-> create Argo CD project
-> prepare Helm files
-> update branch
-> push files to Git
-> deploy bootstrap chart
-> Argo CD syncs
-> ACM provisions
-> cluster comes online
```

---

## 29. Troubleshooting for Beginners

### Problem 1 - Git authentication fails

```bash
git config --global credential.helper cache
```

If your environment uses another credential tool, follow your team standard.

### Problem 2 - Git commit fails

```bash
git config --global user.email "your_email@example.com"
git config --global user.name "First LastName"
```

### Problem 3 - Secret import fails
Check:

- `metadata.name`
- `metadata.namespace`
- YAML indentation
- `type: Opaque`
- base64 formatting in `data`

### Problem 4 - Git branch is wrong

```bash
git branch --show-current
```

Also verify `targetRevision` in `values.yaml`.

### Problem 5 - `oc get nodes` fails
Check:

- login command
- cluster URL
- token validity
- network access from bastion

### Problem 6 - Argo CD app is broken
Check:

- repo URL
- repo credentials
- branch name
- Helm values
- missing secrets

### Problem 7 - Helm install fails
Check:

- chart path
- release name
- values file changes
- cluster connectivity

### Problem 8 - Cluster does not start installing
Check:

- BMC credentials
- pull secret
- host MAC addresses
- host IP values
- gateway
- VLAN
- API VIP
- ingress VIP
- base domain

### Problem 9 - Cluster resources are missing or failing

Check the correct namespace:

```text
cluster-install-secrets
```

Look for:

- AgentClusterInstall
- BareMetalHost
- ClusterDeployment
- InfraEnv
- NMStateConfig

---

## 30. Safe Practice Tips for Beginners

Always:

- work in a test or lab environment first
- double-check IP addresses
- double-check hostnames
- do not delete secrets unless you understand them
- save your work in Git often
- commit with clear messages
- ask before using production credentials

---

## 31. Recommended GitHub Project Structure

```text
openshift-gitops-cluster-platform/
├── README.md
├── docs/
│   ├── guide.md
│   ├── architecture.md
│   ├── troubleshooting.md
│   └── glossary.md
├── helm-charts/
│   ├── cluster-install/
│   │   ├── Chart.yaml
│   │   ├── templates/
│   │   ├── allHosts.yaml
│   │   ├── globalValues.yaml
│   │   └── values/
│   │       └── ocp-lab-01.yaml
│   └── cluster-install-bootstrap/
│       ├── Chart.yaml
│       ├── templates/
│       └── values.yaml
├── scripts/
│   ├── validate-values.sh
│   ├── render-helm.sh
│   └── deploy-bootstrap.sh
├── examples/
│   ├── sample-allHosts.yaml
│   ├── sample-globalValues.yaml
│   └── sample-cluster-values.yaml
└── .github/
    └── workflows/
        └── lint.yml
```

---

## 32. Beginner Build Plan

### Day 1
- Learn Git basics
- Learn what OpenShift is
- Learn what Argo CD does

### Day 2
- Create namespace and secrets
- Open Argo CD and learn the menus

### Day 3
- Connect Git repo
- Clone repo on bastion

### Day 4
- Update `allHosts.yaml`
- Update `globalValues.yaml`

### Day 5
- Create your cluster values file
- Update branch-specific bootstrap values

### Day 6
- Run `helm template`
- Commit and push changes

### Day 7
- Deploy bootstrap chart
- Monitor cluster installation

---

## 33. Summary

You now have a complete beginner roadmap for deploying an OpenShift cluster using custom Helm charts with GitOps and CVI secret creation.

You learned how to:

- prepare the hub cluster
- create and copy secrets
- create an Argo CD project
- connect a Git repository
- prepare Helm values files
- update the correct branch
- generate templates
- push changes to Git
- deploy the bootstrap chart
- monitor the installation
- troubleshoot common problems

---

## 34. Recommended Notes to Capture Each Time

For every execution, document:

- site name
- branch used
- secret name created
- namespace used
- bastion used
- Helm release name
- deployment result
- blockers encountered

This makes future repeat runs easier and reduces troubleshooting time.

---

## 35. Final Encouragement

Take it one step at a time.

You do **not** need to understand everything on day one.

You only need to do this:

1. follow the steps
2. keep your files organized
3. review your values carefully
4. ask questions when something does not look right

That is how strong DevOps and OpenShift engineers grow.
