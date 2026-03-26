# Storage Configuration Runbook

## Purpose

Use this runbook to configure storage networking and validate storage readiness for a new OpenShift cluster through a GitOps-driven workflow.

## Scope

This runbook covers:

- updating storage network YAML inputs
- generating required labels or values for automation
- synchronizing the resulting configuration through GitOps
- validating storage network outcomes
- creating OpenShift Virtualization test VMs to confirm storage behavior

## Prerequisites

- access to the automation Git repository
- SSH access to a bastion host
- access to Argo CD or the approved GitOps dashboard
- target cluster access with `oc`
- approved storage VLAN IDs, IP ranges, MTU, and interface mappings
- secret name for the storage backend or CSI credentials

## Sanitized Example Inputs

| Item | Sanitized example |
|---|---|
| Cluster name | `ocp-storage-lab-01` |
| Branch | `cluster-init` |
| YAML file | `new-cluster.yaml` |
| Storage secret | `storage-backend-secret` |
| Bastion host | `bastion.lab.example.com` |

## Procedure

### 1. Open The Automation Repository

Example:

```text
https://git.example.com/platform/cluster-automation
```

### 2. Switch To The Working Branch

Example:

```text
cluster-init
```

### 3. Locate The Storage Configuration Area

Example path:

```text
policies/storage/files
```

### 4. Connect To The Bastion Host

```bash
ssh username@bastion.lab.example.com
```

### 5. Move To The Storage Networking Directory

```bash
cd storage-cluster-networking
```

### 6. Copy A Template YAML File

```bash
cp example-cluster.yaml new-cluster.yaml
```

### 7. Update Cluster Storage YAML

#### Hostnames

```yaml
hostnames:
  - node01
  - node02
  - node03
```

#### Storage VLANs And IPs

```yaml
vlans:
  - id: 100
    ips:
      - 10.10.10.54
      - 10.10.10.55
      - 10.10.10.56
  - id: 200
    ips:
      - 10.20.20.54
      - 10.20.20.55
      - 10.20.20.56
```

#### MTU And Interface Definitions

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

Always verify that interface names match the actual node inventory.

### 8. Generate Labels Or Derived Configuration

```bash
sh generate-labels.sh new-cluster.yaml
```

Expected output example:

```text
new-cluster-network-labels.yaml
```

### 9. Update The Values File Used By Automation

Open the values file and paste the generated labels or structured configuration into the required section.

Example:

```text
values.yaml
```

### 10. Configure Storage Operator Inputs

```yaml
storage:
  enabled: true
  operator-name: storage-operator
  install-mode: automatic
  operator-source: certified-operators
  operator-channel: stable
  storage-credentials-secret: storage-backend-secret
```

### 11. Sync Through GitOps

Trigger synchronization in the GitOps system and wait until the storage-related applications or policies report healthy and synced.

## Validation

### Configuration Validation

Confirm:

- VLAN IDs match the approved network design
- hostnames map to the correct nodes
- MTU values are expected on the target interfaces
- storage credential secret exists in the expected namespace

Useful checks:

```bash
oc get nncp -A
oc get nnce -A
oc get pods -n openshift-storage
oc get storageclass
oc get pvc -A
```

### Validate Storage Using OpenShift Virtualization VMs

If OpenShift Virtualization is installed, create one or more test VMs to validate that storage classes, PVC provisioning, and network-backed workloads operate correctly.

#### Example DataVolume

```yaml
apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
  name: rhel9-test-dv
  namespace: openshift-cnv
spec:
  source:
    blank: {}
  pvc:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 40Gi
    storageClassName: ocs-external-storagecluster-ceph-rbd
```

#### Example VirtualMachine

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: rhel9-storage-validation
  namespace: openshift-cnv
spec:
  running: false
  template:
    metadata:
      labels:
        app: rhel9-storage-validation
    spec:
      domain:
        cpu:
          cores: 2
        devices:
          disks:
            - name: rootdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
        resources:
          requests:
            memory: 4Gi
      networks:
        - name: default
          pod: {}
      volumes:
        - name: rootdisk
          dataVolume:
            name: rhel9-test-dv
        - name: cloudinitdisk
          cloudInitNoCloud:
            userData: |
              #cloud-config
              user: cloud-user
              password: <redacted>
              chpasswd:
                expire: false
```

#### VM Validation Steps

1. Create the `DataVolume` and wait for the PVC to bind.
2. Create the `VirtualMachine`.
3. Start the VM and confirm it reaches the running state.
4. Verify the PVC was provisioned by the expected storage class.
5. If required, attach an additional test disk and verify read and write operations inside the guest.

Useful commands:

```bash
oc get dv,pvc,vm,vmi -n openshift-cnv
oc describe pvc rhel9-test-dv -n openshift-cnv
virtctl start rhel9-storage-validation -n openshift-cnv
```

## Best Practices

- treat interface names and MTU values as host-specific facts to be verified, not assumed
- keep storage and network changes in the same review only when they are operationally coupled
- validate generated labels before syncing GitOps changes
- confirm the target storage class and backend secret names from the cluster, not from stale notes
- use test PVCs or VMs to validate provisioning before application cutover
- keep all examples sanitized and environment-neutral in shared docs

## Troubleshooting

### GitOps Sync Succeeds But Storage Pods Fail

- inspect `openshift-storage` pod events and logs
- confirm the storage operator channel and source are valid for the cluster version
- verify the backend credential secret exists and is readable

### PVCs Stay Pending

- check whether the target `StorageClass` exists
- inspect the provisioner name and events on the PVC
- confirm the backend has capacity and network reachability

### Node Network Configuration Does Not Apply

- compare interface names in YAML with the real node interfaces
- inspect `NodeNetworkConfigurationPolicy` and enactment status
- verify MTU and VLAN settings are valid for the physical switch configuration

### Virtual Machine Validation Fails

- confirm OpenShift Virtualization and CDI are installed
- verify the selected storage class supports the requested access mode
- inspect the `DataVolume`, PVC, VM, and VMI events for provisioning failures

### Escalation Data To Capture

Capture:

- Git commit SHA
- cluster name
- storage class name
- failing PVC or VM name
- relevant operator pod names
- first error from PVC, DV, or NNCP events
