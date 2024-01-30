# Failure Domain (FD) Support for Control-Plane Nodes

## Summary

CAPI supports failure domains (FDs) for control-plane nodes via the KCP
controller. CAPI will look for the set of FailureDomains in the
`ProviderCluster.Spec.FailureDomain`, if specified, the field will be copied
to the `Cluster.Status.FailureDomains`. KCP will choose an FD from this set
and place its value in `Machine.Spec.FailureDomain`. Currently, KCP tries
to balance Machines evenly across all FDs.

It is expected that providers will use this chosen FD (in the Machine.Spec)
in deciding where to place the infrastructure machine. In the case of
metal3, CAPM3 should associate the Metal3Machine with the corresponding BMH
in the desired FD.

In public clouds FailureDomains are well defined. For example, in Azure, FDs
map to availability zones within an azure region. For baremetal, there is no
such mapping. However, an operator of a baremetal/metal3 cluster may still
place their hosts across FDs in their data center or region.
For example, two racks, connected to independent power grids, may
constitute two FDs. Labels placed on the BMHs, by the operator, can provide a
means to distinguish between hosts belonging to different FDs.

The proposal is for CAPM3 to choose hosts based on the FD selected by KCP.
In other words, based on the value in `Machine.Spec.FailureDomain` CAPM3 will
choose a BMH matching the label corresponding to that FD.

## User Stories

As an operator who has placed their baremetal infrastructure across different
FDs, I would like CAPM3 to associate control-plane nodes with BMHs from the
desired FD.

## Goals

* Support selection of BMHs based on the desired FD for the control plane node.

## Non-Goals

* Failure Domains support for worker nodes (i.e. MachineDeployments).

## Proposal

The operator labels the BMH resource with a standard FD label. For
example, the following standard label could be used on the BMH:

```diff
  infrastructure.cluster.x-k8s.io/failure-domain=my-fd-1
```

Today, CAPM3 chooseHost() func associates a Metal3Machine with a specific BMH
based on labels specified in `Metal3Machine.Spec.HostSelector.MatchLabels`.
We can expand this capability. The HostSelector field is used to narrow down
the set of available BMHs that meet the selection criteria. When FDs are
being utilized (i.e. `Machine.Spec.FailureDomain` != nil and Machine is for a
control-plane node), we insert the above label into `HostSelector.MatchLabels`.
This should, in effect, narrow down the set of available BMHs to only those
in the desired FD. Additional labels in the set could further narrow down the
set of usable BMHs.

Note that if no FD is specified in the `Machine.Spec.FailureDomain`, then the
chooseHost() will default to it's current behavior.

### Example Scenario

Consider the following scenario. An operator has FDs: fd-1 and fd-2. These are
specified in Metal3Cluster:

```yaml
kind: Metal3Cluster
spec:
  failureDomains:
    fd-1:
      controlPlane: true
    fd-2:
      controlPlane: true
...
```

When a control-plane node needs to be created, KCP will choose an FD from this set
and place its value in `Machine.Spec.FailureDomain`. Let's assume that KCP has
selected fd-1 for a new control-plane node:

```yaml
kind: Machine
name: mycluster-control-plane-7wrls
spec:
  failureDomain: fd-1
...
```

The operator has added the following label to a host in fd-1:

```yaml
kind: BareMetalHost
name: some-bmh-in-fd-1
metadata:
  labels:
    infrastructure.cluster.x-k8s.io/failure-domain: fd-1
...
```

When CAPM3 reconciles the `Metal3Machine` object representing the control-plane
Machine that KCP created, it will check if `Machine.Spec.FailureDomain` has a value,
which in our example is fd-1. CAPM3 will pick a BMH that has the fd-1 label, such as
the one we have defined above.

## FD labels on the Node object

Kubernetes has the concept of [zones and regions.](https://kubernetes.io/docs/setup/best-practices/multiple-zones/)
A zone can be considered an FD and 2+ zones fit within a logical grouping
called a region. Nodes can be labeled with the well-known [zone label:](https://kubernetes.io/docs/reference/labels-annotations-taints/#topologykubernetesiozone)

```diff
    topology.kubernetes.io/zone=my-fd-1
    topology.kubernetes.io/zone=my-fd-2
```

For the purpose of scheduling pods (and placing pvcs), the zone labels can
be utilized by the kubernetes scheduler to spread workloads across
different FDs. While this proposal is only geared towards control-plane
nodes, and zone labels are mostly applicable for worker nodes, we discuss
some options here for operators who wish to have these labels applied to
their control-plane nodes.

### BMH Label Sync

CAPM3 supports a [label sync mechanism](https://github.com/metal3-io/metal3-docs/blob/main/design/sync-labels-bmh-to-node.md)
between the BMH and its corresponding Kubernetes Node. The sync controller
could be configured to sync zone labels
(e.g.`topology.kubernetes.io/zone=my-fd-1`) placed on the BMH to the
corresponding Node. The operator would add this label to the BMH in
addition to standard FD label described above
(ie. `infrastructure.cluster.x-k8s.io/failure-domain=<my-fd-1>`).

### CAPI Support

The label `topology.kubernetes.io/zone` is [*not* restricted](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction),
so it can be set by the Kubelet `--node-labels` flag. CAPI could insert
this label via CABPK when it generates the kubeadm config for a
control-plane node. At the time of this writing, there is currently an open
issue for [this feature](https://github.com/kubernetes-sigs/cluster-api/issues/5667).

### Cloud-Provider-Interface

In public clouds, these label are generally added via the
[cloud-provider-interface (CPI).](https://github.com/kubernetes/cloud-provider)
It's the cloud-provider's job of initializing a node with cloud specific
zone/region labels. For more advanced use case, operators may implement
a cloud-provider of their own.

## Related

* [CAPV FD support](https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/blob/master/docs/proposal/20201103-failure-domain.md)
