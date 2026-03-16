# Ansible Automation Platform (AAP)

## Beginner Guide to Creating an OpenShift Cluster with Automation Templates

Author: Emmanuel Naweji\
Audience: Complete Beginners

------------------------------------------------------------------------

# 1. What is Ansible Automation Platform?

Ansible Automation Platform (AAP) is a web-based system used to automate
IT operations.

With AAP you can:

-   Deploy infrastructure
-   Configure servers
-   Run automation playbooks
-   Manage clusters
-   Execute automation templates

Everything is managed through a web interface.

------------------------------------------------------------------------

# 2. High Level Workflow

User → AAP Web Interface → Automation Template → Provide Inputs →
Cluster Creation

------------------------------------------------------------------------

# 3. Step 1 --- Access the AAP Platform

Open your browser and go to the AAP URL.

Example:

https://automation.example.com

You will see the login page.

------------------------------------------------------------------------

# 4. Step 2 --- Login

Enter your credentials.

Example:

Username: your-username\
Password: your-password

Click **Login**.

------------------------------------------------------------------------

# 5. Step 3 --- Navigate to Automation Execution

Inside the platform:

Automation Execution → Templates

Templates are prebuilt automation workflows.

------------------------------------------------------------------------

# 6. Step 4 --- Select a Template

Find the template used to create a cluster.

Example:

OpenShift Cluster Install

Click **Launch Template**.

------------------------------------------------------------------------

# 7. Step 5 --- Gather Required Inputs

Before continuing, gather the following information.

  Item                   Description
  ---------------------- ----------------------------
  Cluster Name           Name of the cluster
  Default Gateway        Network gateway
  API Virtual IP         IP used for Kubernetes API
  Ingress / Apps VIP     IP used for applications
  VLAN ID                Network VLAN
  Machine Network CIDR   Machine network range

------------------------------------------------------------------------

# 8. Step 6 --- Fill Out Credentials

Select credentials from the dropdown menu.

Examples:

-   vCenter Credentials
-   OpenShift Credentials
-   GitHub Credentials

------------------------------------------------------------------------

# 9. Step 7 --- Enter Node Information

Define cluster nodes.

Example:

Control Plane Nodes\
Worker Nodes

Example values:

FQDN: node1.example.com\
Serial: OITANNVSH60S\
IP: 10.93.44.9

------------------------------------------------------------------------

# 10. Step 8 --- Provide Network Information

Example configuration:

Cluster Name: dev-cluster

Default Gateway: 10.93.44.1

API Virtual IP: 10.93.44.10

Ingress / Apps VIP: 10.93.44.11

VLAN ID: 120

Machine Network CIDR: 10.93.44.0/24

------------------------------------------------------------------------

# 11. Step 9 --- Click Next

Click **Next**, review the configuration, then click **Finish**.

------------------------------------------------------------------------

# 12. What Happens Next?

AAP will:

1.  Generate cluster configuration
2.  Create a GitHub repository
3.  Store installation files
4.  Execute Ansible playbooks

------------------------------------------------------------------------

# 13. Example GitHub Repository Structure

dcce/ └── openshift-cluster-install/ └── ansible/ └── clusters/ └──
odet-v-w-1/

------------------------------------------------------------------------

# 14. Important Configuration Files

### allhosts.yml

Defines cluster nodes.

``` yaml
all:
  hosts:
    master1:
      ansible_host: 10.93.44.20
    worker1:
      ansible_host: 10.93.44.30
```

### allhosts_by_hostname.yml

Groups nodes.

``` yaml
masters:
  - master1
workers:
  - worker1
```

### prefix_network.yml

Network configuration.

``` yaml
machineNetwork:
  - cidr: 10.93.44.0/24

apiVIP: 10.93.44.10
ingressVIP: 10.93.44.11
```

### values.yaml

Global values.

``` yaml
clusterName: dev-cluster
vlanID: 120
defaultGateway: 10.93.44.1
```

------------------------------------------------------------------------

# 15. Automation Flow

Template Launch → Gather Inputs → Generate Repo → Create Inventory → Run
Playbooks → Install Cluster

------------------------------------------------------------------------

# 16. Checking Job Status

Inside AAP:

Automation Execution → Jobs

You can see:

-   Running jobs
-   Completed jobs
-   Failed jobs

------------------------------------------------------------------------

# 17. Troubleshooting

Check:

-   Credentials
-   Network CIDR
-   IP ranges
-   VLAN configuration
-   Node hostnames

Review logs in the Jobs section.

------------------------------------------------------------------------

# 18. Best Practices

Always:

-   Use clear cluster names
-   Validate IP ranges before deployment
-   Store configurations in Git
-   Use version control

------------------------------------------------------------------------

# Summary

This guide showed how to:

-   Access Ansible Automation Platform
-   Launch automation templates
-   Provide cluster configuration
-   Generate GitHub configuration files
-   Install an OpenShift cluster automatically
