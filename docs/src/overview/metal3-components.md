# Metal³ Components

## Baremetal Operator

Unlike any other Cluster API providers, Metal³ comes with an
extra component called [Baremetal Operator](https://github.com/metal3-io/baremetal-operator)
(BMO), which manages the actual bare metal hosts. The reason for
introducing an extra component is because all the other cloud
providers already had built APIs to manage their respective
infrastructure whereas in Metal³ there was no baremetal
infrastructure APIs.

### Ironic integration

Under the hood Baremetal Operator is using OpenStack
[Ironic](https://docs.openstack.org/ironic/latest/) as
a provisioning tool to manage the bare metal hosts. Baremetal
Operator doesn’t require you to deploy and manage the Ironic
since it is the job of Metal³ to manage it.

Please note that, although Baremetal Operator utilizes Ironic
from the OpenStack community, it doesn’t bring any other OpenStack
services into your cluster apart from the bare minimum
[Ironic setup](https://github.com/metal3-io/metal3-docs/blob/master/design/use-ironic.md).

## Cluster-api-provider-metal³

[Cluster-api-provider-metal³](https://github.com/metal3-io/cluster-api-provider-metal3)
(CAPM3) is a provider implementation of the Cluster API. CAPM3
provides Kubernetes APIs to create, configure and manage clusters
on bare metal based environments.