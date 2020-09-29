# Metal³-Dev-Env

In this sectons we will cover the basics of how to run and use
[Metal³-dev-env](https://github.com/metal3-io/metal3-dev-env)
to create a Kubernetes cluster in Metal³ development environment.
Metal³-dev-env is an emulated environment which spins up a set
of Virtual Machines (VM)s and does the management of those VMs
as if they were bare metal hosts.

## Installation

### Prerequisites

- System with CentOS 8 or Ubuntu 20.04 (Focal Fossa)
- System with at least 4C CPUs, 16 GB RAM memory.
- Run as a user with passwordless sudo access
- Export a couple of environment variables if you don’t want to 
use the [default ones](https://github.com/metal3-io/metal3-dev-env/blob/master/vars.md)

```shell
export IMAGE_OS=Ubuntu
export CONTAINER_RUNTIME=docker
export EPHEMERAL_CLUSTER=kind
```

Either [kind](https://kind.sigs.k8s.io/) or [minikube](https://minikube.sigs.k8s.io/docs/start/)
can be used for creating a bootstrap
Kubernetes cluster. By default for Ubuntu based systems,
Metal³-dev-env will use kind to create a bootstrap cluster
while for CentOS based systems it is a Minikube cluster. To
manipulate cluster creation tool use  `EPHEMERAL_CLUSTER` variable:

```shell
export  EPHEMERAL_CLUSTER=<...>
```

To define what OS distro should be used for the target
cluster nodes please export the `IMAGE_OS` environment
variable with desired value. Possible options are Ubuntu
and Centos.

```shell
export  IMAGE_OS=<...>
```

### Setup

Now that we have exported necessary environment variables,
we can run make which will execute a series of scripts.
You can find more information [here](https://metal3.io/try-it.html#12-metal3-dev-env-setup)
about what each script is responsible for.

```shell
git clone https://github.com/metal3-io/metal3-dev-env.git
cd /metal3-dev-env
make
```

This setup will take approx. 30min, and during that time the scripts will:

- install necessary packages and tools
- make networking configurations
- create libvirt VMs
- create a bootstrap cluster (kind or minikube)
- start necessary containers (ironic, ironic-inspector, vbmc, sushy, dnsmasq, etc.)
- deploy Cluster API and Metal³ provider components, like CAPM3 and BMO
- apply `BareMetalHost` Custom Resources (CR)s

By the end of the successful run, you should be able to see
`BareMetalHost` objects in metal3 namespace in Ready state.

### Provision cluster and machines

Now that we have `BareMetalHost` objects in `Ready` state, 
we can start provisioning them. 