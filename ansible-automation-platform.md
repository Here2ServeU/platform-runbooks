# Ansible Automation Platform (AAP)
## Beginner Guide to Creating an OpenShift Cluster with Automation Templates

**Author:** Emmanuel Naweji  
**Audience:** Complete beginners  
**Use case:** Documenting how to launch an OpenShift cluster installation workflow from Ansible Automation Platform and store the generated configuration in GitHub.

---

## 1. What is Ansible Automation Platform?

Ansible Automation Platform (AAP) is a web-based platform used to automate IT work.

With AAP, teams can:

- deploy infrastructure
- configure servers
- run automation playbooks
- manage cluster installation workflows
- standardize repeatable operations

For this guide, you will use AAP to launch an **OpenShift cluster installation template**.

---

## 2. High-Level Workflow

The process is simple:

1. Open the AAP web portal
2. Log in
3. Go to **Automation Execution**
4. Open **Templates**
5. Launch the OpenShift cluster installation template
6. Fill in credentials and cluster details
7. Submit the job
8. Let AAP generate the configuration and GitHub repo content

---

## 3. Step 1 — Open the AAP URL

Open your browser and go to your company's AAP web address.

Example:

```text
https://aap.example.internal
```

You should see the login page.

---

## 4. Step 2 — Log In

Enter your username and password.

Example:

```text
Username: your-username
Password: your-password
```

Then click **Login**.

---

## 5. Step 3 — Open Automation Execution

After logging in:

- go to **Automation Execution**
- click **Templates**

This is where reusable automation jobs are stored.

---

## 6. Step 4 — Launch the Cluster Template

Find the template for OpenShift cluster creation.

Example template name:

```text
OpenShift Cluster Install
```

Then click **Launch Template**.

---

## 7. Step 5 — Gather the Information You Need

Before filling in the form, collect the values below.

### Required inputs

| Field | What it means |
|---|---|
| Cluster Name | Name of the new cluster |
| Default Gateway | Main route out of the network |
| API Virtual IP | Virtual IP for OpenShift API |
| Ingress / Apps Virtual IP | Virtual IP for application traffic |
| VLAN ID | Network VLAN number |
| Machine Network CIDR | IP range used by machines |
| Node Hostnames | Names of control plane and worker nodes |
| Node IPs | IP addresses of the nodes |
| Credentials | Access credentials from dropdown menus |

---

## 8. Step 6 — Select Credentials

In the template form, choose the required credentials from the dropdown menus.

Examples:

- vCenter credentials
- OpenShift credentials
- GitHub credentials
- automation service account credentials

Choose the correct credentials for your environment.

---

## 9. Step 7 — Enter Node Information

You will usually need node details for the cluster.

Typical node groups:

- control plane nodes
- worker nodes

### Example node data

> These are sample values using random private IP addresses for documentation only.

| Role | Hostname | Serial | IP Address |
|---|---|---|---|
| Control Plane | ocp-cp-01.lab.example | SN-A91K2P7 | 10.42.18.21 |
| Control Plane | ocp-cp-02.lab.example | SN-B73M8Q4 | 10.42.18.22 |
| Control Plane | ocp-cp-03.lab.example | SN-C55R1T9 | 10.42.18.23 |
| Worker | ocp-wk-01.lab.example | SN-D20L6X8 | 10.42.18.31 |
| Worker | ocp-wk-02.lab.example | SN-E84N3Z1 | 10.42.18.32 |

---

## 10. Step 8 — Enter Network Information

Fill in the network section carefully.

### Example values

```text
Cluster Name: ocp-lab-cluster-01
Default Gateway: 10.42.18.1
API Virtual IP: 10.42.18.10
Ingress / Apps Virtual IP: 10.42.18.11
VLAN ID: 218
Machine Network CIDR: 10.42.18.0/24
```

Make sure these values match your network design.

---

## 11. Step 9 — Review and Submit

After entering the information:

1. Click **Next**
2. Review the summary page
3. Click **Finish**

This starts the automation job.

---

## 12. What Happens After You Click Finish?

AAP will typically do the following:

1. validate the inputs
2. generate cluster configuration files
3. create or update content in a GitHub repository
4. prepare inventory and network files
5. run the installation workflow

---

## 13. Better OpenShift GitHub Project Structure

Below is a cleaner project structure you can use in GitHub for your OpenShift automation project.

```text
openshift-cluster-install/
├── README.md
├── docs/
│   ├── guide.md
│   ├── architecture.md
│   ├── installation-workflow.md
│   └── troubleshooting.md
├── ansible/
│   ├── inventories/
│   │   └── ocp-lab-cluster-01/
│   │       ├── hosts.yml
│   │       ├── hosts_by_hostname.yml
│   │       ├── network.yml
│   │       └── values.yml
│   ├── playbooks/
│   │   ├── validate-inputs.yml
│   │   ├── generate-config.yml
│   │   └── install-cluster.yml
│   ├── roles/
│   │   ├── cluster_validation/
│   │   ├── network_config/
│   │   └── openshift_install/
│   └── group_vars/
│       └── all.yml
├── scripts/
│   ├── launch-template.sh
│   ├── verify-config.sh
│   └── post-install-check.sh
├── examples/
│   └── sample-cluster-inputs.md
└── .github/
    └── workflows/
        └── lint-ansible.yml
```

### Why this structure is better

- **README.md** explains the project
- **docs/** keeps all documentation in one place
- **ansible/inventories/** separates each cluster cleanly
- **playbooks/** stores executable automation steps
- **roles/** keeps the automation modular
- **scripts/** stores helper commands
- **examples/** gives beginners sample input files
- **.github/workflows/** helps automate quality checks

---

## 14. Important Configuration Files

These are the kinds of files your automation may generate.

### `hosts.yml`

Defines the cluster nodes and their IP addresses.

```yaml
all:
  children:
    control_plane:
      hosts:
        ocp-cp-01.lab.example:
          ansible_host: 10.42.18.21
        ocp-cp-02.lab.example:
          ansible_host: 10.42.18.22
        ocp-cp-03.lab.example:
          ansible_host: 10.42.18.23
    workers:
      hosts:
        ocp-wk-01.lab.example:
          ansible_host: 10.42.18.31
        ocp-wk-02.lab.example:
          ansible_host: 10.42.18.32
```

### `hosts_by_hostname.yml`

Groups systems by hostname lists.

```yaml
control_plane:
  - ocp-cp-01.lab.example
  - ocp-cp-02.lab.example
  - ocp-cp-03.lab.example

workers:
  - ocp-wk-01.lab.example
  - ocp-wk-02.lab.example
```

### `network.yml`

Stores network settings.

```yaml
cluster_name: ocp-lab-cluster-01
default_gateway: 10.42.18.1
api_vip: 10.42.18.10
ingress_vip: 10.42.18.11
vlan_id: 218
machine_network_cidr: 10.42.18.0/24
```

### `values.yml`

Stores additional values used by the automation.

```yaml
cluster_name: ocp-lab-cluster-01
environment: lab
domain: lab.example
platform: vmware
```

---

## 15. Example Automation Flow

```text
User opens AAP
   ↓
User launches OpenShift template
   ↓
User fills in credentials and cluster values
   ↓
AAP validates inputs
   ↓
AAP generates inventory and network files
   ↓
AAP updates GitHub project content
   ↓
AAP runs installation workflow
   ↓
OpenShift cluster deployment begins
```

---

## 16. How to Check Job Status

Inside AAP:

- go to **Automation Execution**
- open **Jobs**

There you can see:

- running jobs
- successful jobs
- failed jobs

Click on a job to read its logs.

---

## 17. Troubleshooting Tips for Beginners

If the job fails, check these first:

- wrong credentials selected
- duplicate or invalid IP addresses
- incorrect VLAN ID
- wrong gateway
- wrong machine network CIDR
- hostname spelling mistakes
- required fields left empty

Also check the job log in AAP to see exactly where it failed.

---

## 18. Best Practices

Always try to do the following:

- use clear cluster names
- document every network value
- keep sample configs in GitHub
- separate docs from automation files
- review inputs before clicking **Finish**
- use version control for every change

---

## 19. Summary

In this guide, you learned how to:

- access Ansible Automation Platform
- log in and open templates
- launch an OpenShift cluster automation template
- enter credentials and network values
- organize the generated content in a cleaner GitHub project structure
- understand the key files used for cluster installation

---

## 20. Final Note

AAP helps teams automate complex work with a repeatable process.

That means:

- fewer manual mistakes
- faster setup
- easier documentation
- more consistent OpenShift deployments

For beginners, the key is simple:

**fill in the right values, review carefully, and let automation do the heavy work.**
