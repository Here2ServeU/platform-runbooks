# OpenShift Cluster Install Runbook

## Goal
Deploy an OpenShift cluster using:
- AAP (Ansible Automation Platform)
- GitHub (config)
- DevSpaces (execution)
- ACM (cluster management)
- ArgoCD (GitOps deployment)

---

## Flow
AAP → GitHub → DevSpaces → ACM → ArgoCD → Cluster Ready

---

## STEP 1 — AAP: Generate Cluster Config

Go to:
- Ansible Automation Platform → Templates

Do:
- Select template
- Click Launch Template
- Fill required values

Result:
- Generates config
- Creates new Git branch

Example:
abcx-lab-cluster-install-configs.git

---

## STEP 2 — GitHub: Update Config

Go to:
GitHub → repo/randomx-lab-cluster-install-configs

Do:
- Switch to new branch

Update:
cluster-install-bootstrap/
→ values.yaml

cluster-install/

Change:
targetRevision → new branch name

---

## STEP 3 — Launch DevSpaces

Create GitHub token:
Settings → Developer Settings → Tokens (classic)
- Enable SSO

Login DevSpaces:
- Use OpenShift login
- Use repo URL

---

## STEP 4 — Login to ACM

Login:
kubeadmin

Navigate:
Administration → Namespaces

Find:
cluster-install-secrets

---

## STEP 5 — Create Secret

Go to:
Workloads → Secrets

Do:
- Open any secret
- YAML → Copy all

Create new:
+ → Import YAML

Modify:
- Delete lines 1–20
- name → xyz-bmc-credentials
- update site code

Save in:
cluster-install-secrets

---

## STEP 6 — Get Login Command

Copy:
oc login ...

---

## STEP 7 — DevSpaces

Run:
oc login <command>

---

## STEP 8 — Helm Install

cd helm-charts/

Run:
helm install abx-xyz-r-9 cluster-install-bootstrap

---

## STEP 9 — ArgoCD

Go:
ACM → 9 dots → ArgoCD

Login via OpenShift

Apps → All Apps

If Out of Sync:
Click → Sync

---

## STEP 10 — Validate in ACM

Fleet Management → Infrastructure → Host Inventory

Check:
hosts discovered

Then:
Clusters → xyz

---

## STEP 11 — Resource Assignment

Infrastructure → Clusters

Actions → Manage resource assignment → Review → Save

---

## STEP 12 — Governance

Fleet Management → Governance

Check policies

---

## STEP 13 — Access Cluster

Clusters → Web Console URL

Login:
kubeadmin

---

## STEP 14 — Optional Check

Administrator → API Explorer

Project:
trident

---

## Notes

- Ensure correct branch
- Update targetRevision
- Enable SSO for GitHub token
- Sync ArgoCD
