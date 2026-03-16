# Cluster Deployment Using Custom Helm Charts with GitOps
## Complete Beginner Step-by-Step Guide for OpenShift

**Author:** Emmanuel Naweji  
**Format:** Beginner-friendly GitHub documentation  
**Audience:** Complete beginners

---

## 1. What You Are Building

In this guide, you are learning how to deploy an OpenShift cluster using:

- **Git** to store configuration files
- **GitOps** to automatically apply changes from Git
- **Argo CD** to watch the Git repository
- **Helm charts** to package the cluster configuration
- **ACM** to provision and manage the cluster

Think of it like this:

- **Git** = your notebook
- **Helm** = your packaging box
- **Argo CD** = your delivery robot
- **ACM** = the builder
- **OpenShift cluster** = the finished house

---

## 2. Big Picture

Here is the high-level flow:

```text
Secrets -> Git Repo -> Argo CD Project -> Helm Bootstrap -> ACM Provisions -> Cluster Managed
```

This means:

1. You create secrets
2. You connect a Git repository
3. You create a GitOps project
4. You prepare cluster files
5. You deploy the Helm chart
6. ACM creates the cluster
7. The cluster becomes managed

---

## 3. What You Need Before Starting

You should have access to these items:

- OpenShift hub cluster console
- Argo CD / GitOps in OpenShift
- A Git repository
- A bastion host or terminal
- Cluster install values
- Pull secret
- BMC credentials
- SSH public key
- Hostnames, IPs, MAC addresses, VLAN, gateway, and network details

If some of these are given to you by your team, ask for them before starting.

---

## 4. Words You Should Know

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

---

## 5. Step 1 - Prepare the Hub Cluster

Log in to the **Hub cluster OpenShift console**.

You must create a namespace for install secrets.

### Create the namespace

Create this namespace:

```text
cluster-install-secrets
```

This namespace will hold the secrets used during installation.

### Create the required secrets

You need at least these secrets:

1. **cluster-install-pull-secret**
   - used for registry authentication
   - type: image pull secret

2. **bmc-credential**
   - used for BMC, iDRAC, or iLO access
   - contains the username and password for bare metal management

### Simple explanation

- The **pull secret** helps the cluster download required images
- The **BMC secret** helps the platform talk to the physical machines

---

## 6. Step 2 - Create a GitOps Project

Now go to GitOps and create a project.

### Open GitOps settings

In OpenShift GitOps / Argo CD:

```text
Settings -> Projects -> Create Project
```

### Add these items

When creating the project, configure:

- **Source repository**
  - this is the Git repo that stores the cluster install files

- **Destination**
  - allow the hub cluster

- **Resource allow list**
  - allow cluster and namespace resources

### Simple explanation

You are telling Argo CD:

- which repo to trust
- where it is allowed to deploy
- what kinds of resources it is allowed to create

---

## 7. Step 3 - Connect the Git Repository to GitOps

Now connect your Git repository to Argo CD.

### Open Argo CD

From the hub cluster OpenShift console:

1. Go to the top right of the console
2. Open the application menu
3. Select **Infra Argo CD**

### Open repository settings

In Argo CD:

```text
Settings -> Repositories
```

### Add your repository

Enter:

- repository URL
- username
- token or password

### What this does

This gives Argo CD permission to read your Git repo.

---

## 8. Step 4 - Prepare Your Working Folder on the Bastion

Now move to your bastion host or terminal.

### Create a folder

```bash
mkdir git
cd git
```

### Make sure Git authentication works

If your environment uses Git credential manager, sign in using the method your team supports.

Example style from your notes:

```bash
git credential-manager github login --url https://example.git.server
```

Use the real Git server URL from your environment.

---

## 9. Step 5 - Clone the Cluster Install Repository

Clone the repository that holds your cluster-install files.

Example flow:

```bash
git clone <your-repository-url>
cd <your-repository-folder>
```

If your team wants you to work in a different branch:

```bash
git fetch origin source_branch:new_branch
git checkout new_branch
```

### Simple explanation

- `git clone` downloads the repo
- `git checkout` moves you into the branch you will edit

---

## 10. Step 6 - Go to the Helm Chart Folder

Move to the folder that contains the Helm chart for cluster installation.

Example from your notes:

```bash
cd helm-charts/cluster-install
```

This is where the templates and values live.

---

## 11. Step 7 - Update the Main Host File

Find the file named:

```text
allHosts.yaml
```

Update it with your bare metal host details.

### Add these details for each machine

- IP address
- MAC address
- VLAN
- BMC IP
- BMC endpoint
- hostname

### Example beginner-friendly sample

```yaml
hosts:
  - name: master-01
    ip: 10.20.30.21
    mac: "AA:BB:CC:11:22:33"
    vlan: 210
    bmcIP: 10.20.40.21
    bmcEndpoint: "redfish/v1/Systems/System.Embedded.1"

  - name: master-02
    ip: 10.20.30.22
    mac: "AA:BB:CC:11:22:34"
    vlan: 210
    bmcIP: 10.20.40.22
    bmcEndpoint: "redfish/v1/Systems/System.Embedded.1"

  - name: worker-01
    ip: 10.20.30.31
    mac: "AA:BB:CC:11:22:41"
    vlan: 210
    bmcIP: 10.20.40.31
    bmcEndpoint: "redfish/v1/Systems/System.Embedded.1"
```

> These values are sample values for learning. Use your real environment values in production.

---

## 12. Step 8 - Review the Global Values File

Find this file:

```text
globalValues.yaml
```

Review or update these items:

- `clusterSet`
- `secretSourceNamespace`
- `recreateHosts`
- `caTrustBundle`

### What they mean

- **clusterSet** = which group the new cluster will belong to
- **secretSourceNamespace** = where secrets are stored
- **recreateHosts** = whether host resources should be recreated
- **caTrustBundle** = certificate trust data if required

### Example

```yaml
clusterSet: production-east
secretSourceNamespace: cluster-install-secrets
recreateHosts: true
caTrustBundle: ""
```

---

## 13. Step 9 - Create the Cluster-Specific Values File

Create a new file named like this:

```text
clusterName.yaml
```

Replace `clusterName` with the real name of your cluster.

Example:

```text
ocp-lab-01.yaml
```

### This file should include

- cluster name
- base domain
- cluster image set
- BMC credential reference
- pull secret reference
- networking values
- SSH public key

### Example

```yaml
clusterName: ocp-lab-01
baseDomain: lab.example.com
clusterImageSet: img4.16.10-x86-64
bmcCredentialSecret: bmc-credential
pullSecretRef: cluster-install-pull-secret
sshPublicKey: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCexamplekey"
networking:
  machineNetworkCIDR: 10.20.30.0/24
  apiVIP: 10.20.30.10
  ingressVIP: 10.20.30.11
  gateway: 10.20.30.1
  vlan: 210
```

---

## 14. Step 10 - Optional Bootstrap Values Update

If your team uses a different target Git branch, you may need to update:

```text
helm-charts/cluster-install-bootstrap/values.yaml
```

Look for:

```yaml
targetRevision: main
```

Change it if needed.

Example:

```yaml
targetRevision: dev
```

---

## 15. Step 11 - Generate the Helm Template

Run the Helm template command.

Example from your notes:

```bash
helm template ./ -f allHosts.yaml -f globalValues.yaml -f ocp-lab-01.yaml
```

### What this does

This command combines:

- host values
- global values
- cluster-specific values

and turns them into Kubernetes/OpenShift resource definitions.

---

## 16. Step 12 - Review What Changed

Check which files changed.

```bash
git status
```

Add the files:

```bash
git add .
```

Commit your changes:

```bash
git commit -m "Add cluster install config for ocp-lab-01"
```

Push the branch:

```bash
git push
```

### Simple explanation

- `git status` = see what changed
- `git add` = stage your files
- `git commit` = save your change
- `git push` = send it to the Git server

---

## 17. Step 13 - Log In to OpenShift from the Bastion

Now log in to the hub cluster from your bastion.

### Get the login command

In the hub console:

1. Click your username at the top right
2. Select **Copy login command**

### Paste it into the bastion terminal

It will look similar to this:

```bash
oc login --token=sha256~exampletoken --server=https://api.hub.example.com:6443
```

Run it.

---

## 18. Step 14 - Deploy the Helm Chart

Now install the bootstrap Helm chart.

Example from your notes:

```bash
helm install ocp-lab-01 cluster-install-bootstrap
```

Or, depending on your folder structure, run the command from the correct chart directory.

### What happens now

This creates the GitOps application for the cluster install.

You should then see:

- an Argo CD application
- sync activity
- resources being created

---

## 19. Step 15 - Watch the Argo CD Application

After deployment, open Argo CD and check the application.

You want to see:

- app created
- app synced
- app healthy

If it shows degraded or broken, read the error and debug the failing resource.

---

## 20. Step 16 - Monitor Installation Progress

You can monitor installation in several ways.

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

If available, monitor the machine boot sequence using the Out-of-Band IP or BMC console.

---

## 21. What Happens Next

After you deploy, this is the normal flow:

1. Argo CD syncs resources
2. ACM detects a new `ManagedCluster`
3. The nodes boot from ISO or discovery image
4. The cluster install begins
5. The cluster auto-imports into its cluster set
6. The cluster becomes managed

---

## 22. Final Flow in Very Simple Words

```text
Create secrets
-> connect Git repo
-> create Argo CD project
-> prepare Helm files
-> push files to Git
-> deploy bootstrap chart
-> Argo CD syncs
-> ACM provisions
-> cluster comes online
```

---

## 23. Troubleshooting for Beginners

If something breaks, use this checklist.

### Problem 1 - Git authentication fails

Try checking Git credentials.

Examples from your notes:

```bash
git config --global credential.helper cache
```

If your environment uses another credential tool, follow your team standard.

### Problem 2 - Git commit fails

Set your Git identity:

```bash
git config --global user.email "your_email@example.com"
git config --global user.name "First LastName"
```

### Problem 3 - Cluster resources are missing or failing

Check these resources in the API Explorer and make sure you are in the correct namespace:

```text
cluster-install-secrets
```

Look for:

- AgentClusterInstall
- BareMetalHost
- ClusterDeployment
- InfraEnv
- NMStateConfig

### Problem 4 - Argo CD app is broken

Check:

- repo URL
- repo credentials
- branch name
- Helm values
- missing secrets

### Problem 5 - Cluster does not start installing

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

---

## 24. Safe Practice Tips for Beginners

Always do these things:

- work in a test or lab environment first
- double-check IP addresses
- double-check hostnames
- do not delete secrets unless you understand them
- save your work in Git often
- commit with clear messages
- ask before using production credentials

---

## 25. Recommended GitHub Project Structure

Use a project structure like this for your OpenShift GitOps platform:

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

This structure keeps your documentation, Helm charts, scripts, and examples organized.

---

## 26. Beginner Build Plan

If you are building this platform step by step, follow this order:

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

### Day 6
- Run `helm template`
- Commit and push changes

### Day 7
- Deploy bootstrap chart
- Monitor cluster installation

---

## 27. Summary

You now have a simple roadmap for deploying an OpenShift cluster using custom Helm charts with GitOps.

You learned how to:

- prepare the hub cluster
- create secrets
- create an Argo CD project
- connect a Git repository
- prepare Helm values files
- generate templates
- push changes to Git
- deploy the bootstrap chart
- monitor the installation
- troubleshoot common problems

---

## 28. Final Encouragement

Take it one step at a time.

You do **not** need to understand everything on day one.

You only need to do this:

1. follow the steps
2. keep your files organized
3. review your values carefully
4. ask questions when something does not look right

That is how strong DevOps and OpenShift engineers grow.

