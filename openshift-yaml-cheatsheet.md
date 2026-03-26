# OpenShift YAML Cheatsheet

## Purpose

This cheatsheet helps OpenShift engineers write, read, and validate YAML files used for platform operations, GitOps workflows, application delivery, storage validation, and day-two cluster changes.

## How Beginners Should Use This Cheatsheet

If you are new to YAML or OpenShift manifests, use this document in this order:

1. learn what the main YAML building blocks mean
2. learn the common Kubernetes and OpenShift object shape
3. copy a small example and change only one field at a time
4. validate before applying anything to a cluster

Simple meanings of common words:

- `manifest` means a YAML file that describes an object for Kubernetes or OpenShift
- `kind` tells the cluster what type of object you are creating
- `metadata` stores identifying information such as name, namespace, and labels
- `spec` describes the desired state of the object
- `namespace` is a logical boundary inside the cluster, similar to a project

## YAML Fundamentals

### Indentation

Beginner explanation:

- indentation is structure in YAML
- YAML uses spaces to show parent and child relationships
- if indentation is wrong, the file may be invalid or may mean something different from what you intended
- use spaces, not tabs

Example:

```yaml
metadata:
  name: example
  labels:
    app: demo
```

### Scalars

Beginner explanation:

- a scalar is a single value such as text, a number, `true`, `false`, or `null`
- quoting a value can prevent YAML from guessing the wrong type
- IP addresses, versions, and values with special characters are often safer when quoted

Example:

```yaml
stringValue: demo
quotedString: "10.42.18.10"
integerValue: 3
booleanValue: true
emptyString: ""
nullValue: null
```

### Lists

Beginner explanation:

- a list is a set of items under one key
- each list item starts with `-`
- Kubernetes and OpenShift use lists constantly for containers, ports, volumes, and rules

Example:

```yaml
zones:
  - zone-a
  - zone-b
```

### Maps

Beginner explanation:

- a map is a set of key and value pairs
- most Kubernetes objects are large maps containing smaller maps and lists

Example:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 512Mi
```

### Multi-Line Strings

Beginner explanation:

- `|` keeps line breaks exactly as written
- `>` folds multiple lines into one paragraph when parsed
- these styles are useful for scripts, notes, certificates, and cloud-init content

Examples:

```yaml
description: |
  This preserves new lines.
  Useful for scripts, cloud-init, and notes.
```

```yaml
description: >
  This folds lines into a single paragraph
  when parsed.
```

### Multi-Document YAML

Beginner explanation:

- one YAML file can contain more than one object
- `---` separates one object from the next
- this is common when you want to apply related objects together

Example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
  namespace: demo
data:
  key: value
```

### Anchors And Aliases

Beginner explanation:

- anchors let you define a value once and reuse it elsewhere
- aliases reference the anchored value
- some teams avoid them in GitOps repos because they can make files harder for beginners to read

Example:

```yaml
commonLabels: &commonLabels
  app.kubernetes.io/managed-by: gitops
  app.kubernetes.io/part-of: platform

metadata:
  labels: *commonLabels
```

## Kubernetes And OpenShift Object Skeleton

Beginner explanation:

- most manifests start with the same main shape
- `apiVersion` says which API group and version to use
- `kind` says what object type it is
- `metadata` gives the object its identity
- `spec` describes what you want the cluster to do

Example:

```yaml
apiVersion: <group/version>
kind: <Kind>
metadata:
  name: <name>
  namespace: <namespace>
spec:
  {}
```

Common beginner mistakes:

- wrong `apiVersion`
- wrong namespace
- labels not matching selectors
- indentation errors in lists under `spec`

## Namespaces

Beginner explanation:

- a namespace is like a project boundary inside the cluster
- it helps separate applications, teams, and permissions
- many objects are created inside a namespace, so this field matters a lot

Example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: platform-demo
  labels:
    pod-security.kubernetes.io/enforce: baseline
```

Validate:

```bash
oc apply --dry-run=client -f namespace.yaml
```

## ConfigMaps

Beginner explanation:

- a ConfigMap stores non-secret configuration
- use it for values like feature flags, hostnames, log levels, or simple application settings
- do not store passwords or tokens here

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: platform-demo
data:
  LOG_LEVEL: info
  FEATURE_FLAG: "true"
```

## Secrets

Beginner explanation:

- a Secret stores sensitive values such as passwords, tokens, or keys
- `stringData` is easier for humans to write than base64-encoded `data`
- the cluster converts `stringData` into stored secret data
- never commit live secret values to Git

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: platform-demo
type: Opaque
stringData:
  username: <redacted-username>
  password: <redacted-password>
```

## Service Accounts

Beginner explanation:

- a ServiceAccount is an identity used by pods or automation inside the cluster
- it is not the same as a human user account
- workloads use ServiceAccounts when they need permissions to call the Kubernetes API

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: deployer
  namespace: platform-demo
```

## RBAC

Beginner explanation:

- RBAC means role-based access control
- a `Role` defines what actions are allowed
- a `RoleBinding` connects that role to a user, group, or ServiceAccount
- this is how you control who can read, create, update, or delete resources

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: config-reader
  namespace: platform-demo
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: config-reader-binding
  namespace: platform-demo
subjects:
  - kind: ServiceAccount
    name: deployer
    namespace: platform-demo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: config-reader
```

## Deployments

Beginner explanation:

- a Deployment manages one or more identical pods for an application
- `replicas` tells the cluster how many copies you want running
- the selector must match the pod labels, or the Deployment cannot manage the pods correctly
- this is one of the most common object types used to run stateless applications

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: platform-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: quay.io/example/web:1.0.0
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: app-config
```

## Services

Beginner explanation:

- a Service gives a set of pods a stable network identity
- pods come and go, but the Service name stays the same
- `port` is the port the Service exposes
- `targetPort` is the port used by the container

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: platform-demo
spec:
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

## Routes

Beginner explanation:

- a Route is an OpenShift object that exposes a Service externally
- this is how users often reach an application from outside the cluster
- Routes are specific to OpenShift and are commonly used instead of exposing services directly

Example:

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: web
  namespace: platform-demo
spec:
  to:
    kind: Service
    name: web
  port:
    targetPort: http
  tls:
    termination: edge
```

## Jobs And CronJobs

Beginner explanation:

- a Job runs a task once
- a CronJob runs a task on a schedule, similar to cron on Linux
- these are useful for checks, cleanup, backups, and one-time automation

Examples:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: schema-check
  namespace: platform-demo
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: check
          image: registry.access.redhat.com/ubi9/ubi-minimal
          command: ["/bin/sh", "-c", "echo validation"]
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-healthcheck
  namespace: platform-demo
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: healthcheck
              image: registry.access.redhat.com/ubi9/ubi-minimal
              command: ["/bin/sh", "-c", "echo nightly check"]
```

## PersistentVolumeClaims

Beginner explanation:

- a PVC asks the cluster for storage
- the application usually consumes the PVC, not the real storage system directly
- `storageClassName` tells the cluster which storage backend should provide the volume

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
  namespace: platform-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: ocs-external-storagecluster-ceph-rbd
```

## StorageClass Example

Beginner explanation:

- a StorageClass defines how storage should be provisioned
- application teams often use it indirectly through PVCs
- cluster administrators usually create and manage the real StorageClasses

Example:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Cluster administrators should confirm the actual provisioner used in their environment.

## Argo CD Application

Beginner explanation:

- this object tells Argo CD which Git repository path to watch
- it also tells Argo CD where to apply the manifests
- `selfHeal: true` means Argo CD tries to correct drift from the desired state in Git
- this is one of the core objects used in GitOps workflows

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cluster-install
  namespace: openshift-gitops
spec:
  project: platform
  source:
    repoURL: https://github.com/example-org/platform-gitops.git
    targetRevision: main
    path: clusters/ocp-lab-01
  destination:
    server: https://kubernetes.default.svc
    namespace: cluster-your-secrets
  syncPolicy:
    automated:
      prune: false
      selfHeal: true
```

## Helm Values Pattern

Beginner explanation:

- a Helm values file is usually an input file, not the final manifest
- you usually edit values and let Helm render the final YAML
- this helps teams keep repeated configuration clean and reusable

Example:

```yaml
clusterName: ocp-lab-01
baseDomain: lab.example.com
networking:
  machineNetworkCIDR: 10.42.18.0/24
  apiVIP: 10.42.18.10
  ingressVIP: 10.42.18.11
```

Guidance:

- keep values files focused on inputs, not whole rendered manifests
- avoid duplicating defaults already defined in the chart unless you are intentionally overriding them

## MachineConfig Example

Beginner explanation:

- a MachineConfig changes operating system configuration on cluster nodes
- this is powerful and should be handled carefully because it can reconfigure or reboot nodes
- MachineConfig changes are usually part of platform engineering, not normal application deployment

Example:

```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: 99-worker-sysctl
  labels:
    machineconfiguration.openshift.io/role: worker
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
        - path: /etc/sysctl.d/99-worker.conf
          mode: 0644
          overwrite: true
          contents:
            source: data:text/plain;charset=utf-8;base64,bmV0LmlwdjQuaXBfZm9yd2FyZD0xCg==
```

Note: avoid hand-authoring base64 content unless necessary; generate it deterministically.

## Node Network Configuration Policy Example

Beginner explanation:

- this object is used with NMState to configure node networking declaratively
- it can create VLAN interfaces, bonds, and IP settings on selected nodes
- mistakes here can disrupt node connectivity, so validate carefully before applying

Example:

```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: storage-bond0-vlan100
spec:
  desiredState:
    interfaces:
      - name: bond0.100
        type: vlan
        state: up
        vlan:
          base-iface: bond0
          id: 100
        ipv4:
          enabled: true
          dhcp: false
          address:
            - ip: 10.10.10.54
              prefix-length: 24
  nodeSelector:
    node-role.kubernetes.io/worker: ""
```

## KubeVirt VirtualMachine Example

Beginner explanation:

- OpenShift Virtualization uses YAML objects like this to define virtual machines inside the cluster
- the VM references storage, memory, CPU, and networking settings just like other Kubernetes resources
- platform engineers often use these definitions for validation or internal platform workloads

Example:

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-sample
  namespace: openshift-cnv
spec:
  running: false
  template:
    spec:
      domain:
        cpu:
          cores: 2
        devices:
          disks:
            - name: rootdisk
              disk:
                bus: virtio
        resources:
          requests:
            memory: 4Gi
      volumes:
        - name: rootdisk
          persistentVolumeClaim:
            claimName: app-data
```

## OpenShift Patterns Engineers Use Constantly

### Labels And Selectors

Beginner explanation:

- labels are key-value tags attached to objects
- selectors use those labels to find matching objects
- many controller problems come from labels and selectors not lining up

Example:

```yaml
metadata:
  labels:
    app.kubernetes.io/name: api
    app.kubernetes.io/component: backend
    app.kubernetes.io/managed-by: argocd
```

### Resource Requests And Limits

Beginner explanation:

- requests tell the scheduler the minimum resources the container needs
- limits cap how much it is allowed to use
- bad values here can cause poor scheduling, throttling, or instability

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 1
    memory: 1Gi
```

### Probes

Beginner explanation:

- liveness probes tell Kubernetes whether the application is still alive
- readiness probes tell Kubernetes whether it is ready to receive traffic
- these settings help prevent traffic from going to broken or still-starting containers

Example:

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 20
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Tolerations And Node Selectors

Beginner explanation:

- node selectors steer workloads toward nodes with matching labels
- tolerations allow workloads onto nodes with certain taints or scheduling restrictions
- these features are commonly used for infra nodes, storage nodes, or specialized hardware

Example:

```yaml
nodeSelector:
  node-role.kubernetes.io/infra: ""
tolerations:
  - key: node-role.kubernetes.io/infra
    operator: Exists
    effect: NoSchedule
```

## Validation Commands Every OpenShift Engineer Should Know

Beginner explanation:

- validate before applying
- `yamllint` checks formatting and syntax
- `oc apply --dry-run=client` checks whether the manifest is valid without changing the cluster
- `oc diff` shows what would change
- `oc explain` helps you understand valid fields for a resource

Examples:

```bash
yamllint file.yaml
oc apply --dry-run=client -f file.yaml
oc diff -f file.yaml
oc explain deployment.spec.template.spec.containers
oc api-resources | grep -i route
```

## Common YAML Mistakes

Beginner explanation:

- many YAML failures are caused by simple formatting mistakes or wrong references
- if something looks correct but still fails, check spacing, names, and object references before assuming the platform is broken

Common mistakes:

- using tabs instead of spaces
- mixing strings and integers unintentionally
- forgetting quotes around values containing special characters
- mismatched labels and selectors
- wrong `apiVersion` for the cluster version
- bad list indentation under `containers`, `ports`, or `volumes`
- committing live secrets in `data` or `stringData`

## Troubleshooting

### YAML Parses But OpenShift Rejects It

Beginner explanation:

- valid YAML is not always valid Kubernetes or OpenShift YAML
- the file can be structurally correct but still use a wrong field or API version

Checks:

- check the resource schema with `oc explain`
- verify the `apiVersion` exists on the cluster
- inspect whether the namespace or referenced object exists

### A Secret Or ConfigMap Is Referenced But Pods Still Fail

Beginner explanation:

- the object may exist, but the name, namespace, or key could still be wrong
- workloads fail when they cannot find the exact secret or config key they expect

Checks:

- confirm the object name and namespace match exactly
- verify the consuming workload points to the correct key names
- check events on the pod for mount or env injection failures

### Route Or Service Does Not Work

Beginner explanation:

- network exposure problems are often caused by selectors, ports, or missing endpoints
- the Route, Service, and pods all need to line up correctly

Checks:

- verify the Service selector matches pod labels
- confirm the Route target port matches the named Service port
- inspect endpoints with `oc get endpoints`

### PVC Stays Pending

Beginner explanation:

- a pending PVC usually means the cluster could not find suitable storage
- this can be caused by a wrong storage class, unsupported access mode, or backend issue

Checks:

- verify the storage class name
- confirm the requested access mode is supported
- inspect PVC events for provisioner errors

## Practical Rule Of Thumb

If the YAML is long, validate it in three layers:

1. YAML syntax with `yamllint`
2. client-side schema with `oc apply --dry-run=client`
3. rendered outcome or diff with `oc diff` or `helm template`

## Beginner Safe Workflow

Use this flow when editing manifests for the first time:

1. start from a small known-good example
2. change only the name, namespace, or one section at a time
3. run validation commands before applying
4. check the object in the cluster after applying
5. keep notes on what each field changed
