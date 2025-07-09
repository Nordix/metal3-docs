# Using CAPM3 Data Sources

## Resource Relationships Overview

Assuming you've deployed Baremetal Operator and CAPM3 as in a metal3 dev env,
you will likely provision a cluster and control-plane/worker nodes. This process
creates several interconnected resources. The following examples are from the
dev env.

### 1. `Cluster`

The `Cluster` resource includes a reference to the control plane via the
`controlPlaneRef` field:

```yaml
controlPlaneRef:
    apiVersion: controlplane.cluster.x-k8s.io/v1beta1
    kind: KubeadmControlPlane
    name: test1
    namespace: metal3
````

This object defines the cluster and links it to the control plane configuration.

### 2. `KubeadmControlPlane`

This is the main entry point for configuring the control plane. It serves as a
bridge between CAPI and CAPM3. The `KubeadmControlPlane` resource:

- Refers to a `KubeadmConfigTemplate`, which defines kubeadm configuration
- Refers to a `Metal3MachineTemplate`, which defines the infrastructure for
  control plane nodes

```yaml
machineTemplate:
   infrastructureRef:
     apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
     kind: Metal3MachineTemplate
     name: test1-controlplane
     namespace: metal3
```

### 3. `Metal3MachineTemplate` → `Metal3Machine`

The `Metal3MachineTemplate` is used to generate `Metal3Machine` resources during
cluster creation. Each `Metal3Machine`:

- Is linked to a CAPI `Machine` resource
- **Refers to a `Metal3Data`** resource which holds specific node configuration

### 4. `Metal3DataTemplate` → `Metal3Data`

The `Metal3DataTemplate` is a template used to create `Metal3Data` objects.
These objects contain machine-specific configuration, such as:

- Static IP settings
- Network interfaces (NICs)

> **Important**: This is one of the more complex parts of CAPM3
> configuration, because CAPM3 does some processing with the networking data.
> With other templates the data is mostly just copied over to deployed resources
> but the networking template supports convenience features like matching NICs
> via interface name. Misconfiguration here can lead to provisioning errors.
> For example, see: [CAPM3 Issue #1998](https://github.com/metal3-io/cluster-api-provider-metal3/issues/1998)

## Template vs. Concrete Resource

Each key resource in CAPM3 typically has a matching `Template` resource. These
templates are used during provisioning to create real resources, and users
generally only need to configure the templates.

Once these templates are set, CAPM3 will render the actual resources during
cluster creation and updates.
