# Baremetal Operator

Baremetal Operator (BMO) is a custom Kubernetes controller which follows
[Kubernetes operator pattern][Kubernetes operator pattern] and knows how to provision
bare metal machines via `BareMetalHost` Custom Resource (CR). Under the hood the
operator talks to the Openstack [Ironic][Ironic] provisioning service via REST API.
Note that, the operator doesn't bring any other Openstack components in to the cluster.
You can run the operator either within a Kubernetes cluster or outside of the cluster.

## BareMetalHost Custom Resource

Example:

```yaml
apiVersion: metal3.io/v1alpha1
kind: BareMetalHost
metadata:
  name: node-0
spec:
  online: true
  bootMACAddress: 00:34:61:e6:0d:81
  bootMode: legacy
  bmc:
    address: ipmi://192.168.111.1:6230
    credentialsName: node-0-secret
```

<details>
<summary> Example BareMetalHost with inspection data</summary>

```yaml
apiVersion: metal3.io/v1alpha1
kind: BareMetalHost
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"metal3.io/v1alpha1","kind":"BareMetalHost","metadata":{"annotations":{},"name":"node-0","namespace":"metal3"},"spec":{"bmc":{"address":"ipmi://192.168.111.1:6230","credentialsName":"node-0-bmc-secret"},"bootMACAddress":"00:1e:48:1a:26:ba","bootMode":"legacy","online":true}}
  creationTimestamp: "2021-01-22T17:54:06Z"
  finalizers:
  - baremetalhost.metal3.io
  generation: 1
  managedFields:
  - apiVersion: metal3.io/v1alpha1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
      f:spec:
        .: {}
        f:bmc:
          .: {}
          f:address: {}
          f:credentialsName: {}
        f:bootMACAddress: {}
        f:bootMode: {}
        f:online: {}
    manager: kubectl-client-side-apply
    operation: Update
    time: "2021-01-22T17:54:06Z"
  - apiVersion: metal3.io/v1alpha1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .: {}
          v:"baremetalhost.metal3.io": {}
      f:status:
        .: {}
        f:errorCount: {}
        f:errorMessage: {}
        f:goodCredentials:
          .: {}
          f:credentials:
            .: {}
            f:name: {}
            f:namespace: {}
          f:credentialsVersion: {}
        f:hardware:
          .: {}
          f:cpu:
            .: {}
            f:arch: {}
            f:clockMegahertz: {}
            f:count: {}
            f:flags: {}
            f:model: {}
          f:firmware:
            .: {}
            f:bios:
              .: {}
              f:date: {}
              f:vendor: {}
              f:version: {}
          f:hostname: {}
          f:nics: {}
          f:ramMebibytes: {}
          f:storage: {}
          f:systemVendor:
            .: {}
            f:manufacturer: {}
            f:productName: {}
            f:serialNumber: {}
        f:hardwareProfile: {}
        f:lastUpdated: {}
        f:operationHistory:
          .: {}
          f:deprovision:
            .: {}
            f:end: {}
            f:start: {}
          f:inspect:
            .: {}
            f:end: {}
            f:start: {}
          f:provision:
            .: {}
            f:end: {}
            f:start: {}
          f:register:
            .: {}
            f:end: {}
            f:start: {}
        f:operationalStatus: {}
        f:poweredOn: {}
        f:provisioning:
          .: {}
          f:ID: {}
          f:bootMode: {}
          f:image:
            .: {}
            f:url: {}
          f:state: {}
        f:triedCredentials:
          .: {}
          f:credentials:
            .: {}
            f:name: {}
            f:namespace: {}
          f:credentialsVersion: {}
    manager: baremetal-operator
    operation: Update
    time: "2021-01-22T17:59:25Z"
  name: node-0
  namespace: metal3
  resourceVersion: "3391"
  uid: 359edfb0-acbb-4c92-bdaa-246553d806e6
spec:
  bmc:
    address: ipmi://192.168.111.1:6230
    credentialsName: node-0-bmc-secret
  bootMACAddress: 00:1e:48:1a:26:ba
  bootMode: legacy
  online: true
status:
  errorCount: 0
  errorMessage: ""
  goodCredentials:
    credentials:
      name: node-0-bmc-secret
      namespace: metal3
    credentialsVersion: "1775"
  hardware:
    cpu:
      arch: x86_64
      clockMegahertz: 2195
      count: 4
      flags:
      - aes
      - apic
      - arat
      - arch_capabilities
      - avx
      - clflush
      - cmov
      - constant_tsc
      - cpuid
      - cpuid_fault
      - cx16
      - cx8
      - de
      - ept
      - ept_ad
      - erms
      - f16c
      - flexpriority
      - fpu
      - fsgsbase
      - fxsr
      - hypervisor
      - lahf_lm
      - lm
      - mca
      - mce
      - mmx
      - msr
      - mtrr
      - nopl
      - nx
      - pae
      - pat
      - pclmulqdq
      - pge
      - pni
      - popcnt
      - pse
      - pse36
      - pti
      - rdrand
      - rdtscp
      - rep_good
      - sep
      - smep
      - sse
      - sse2
      - sse4_1
      - sse4_2
      - ssse3
      - syscall
      - tpr_shadow
      - tsc
      - tsc_adjust
      - tsc_deadline_timer
      - tsc_known_freq
      - umip
      - vme
      - vmx
      - vnmi
      - vpid
      - x2apic
      - xsave
      - xsaveopt
      - xtopology
      model: Intel Xeon E3-12xx v2 (Ivy Bridge)
    firmware:
      bios:
        date: 04/01/2014
        vendor: SeaBIOS
        version: 1.13.0-1ubuntu1.1
    hostname: node-0
    nics:
    - ip: 192.168.111.20
      mac: 00:1e:48:1a:26:bc
      model: 0x1af4 0x0001
      name: enp2s0
      pxe: false
      speedGbps: 0
      vlanId: 0
    - ip: fe80::2162:77c9:49f9:cf66%enp2s0
      mac: 00:1e:48:1a:26:bc
      model: 0x1af4 0x0001
      name: enp2s0
      pxe: false
      speedGbps: 0
      vlanId: 0
    - ip: 172.22.0.96
      mac: 00:1e:48:1a:26:ba
      model: 0x1af4 0x0001
      name: enp1s0
      pxe: true
      speedGbps: 0
      vlanId: 0
    - ip: fe80::1e80:7a87:6c7e:79d0%enp1s0
      mac: 00:1e:48:1a:26:ba
      model: 0x1af4 0x0001
      name: enp1s0
      pxe: true
      speedGbps: 0
      vlanId: 0
    ramMebibytes: 4096
    storage:
    - hctl: "0:0:0:0"
      model: QEMU HARDDISK
      name: /dev/sda
      rotational: true
      serialNumber: drive-scsi0-0-0-0
      sizeBytes: 53687091200
      vendor: QEMU
    systemVendor:
      manufacturer: QEMU
      productName: Standard PC (Q35 + ICH9, 2009)
      serialNumber: ""
  hardwareProfile: unknown
  lastUpdated: "2021-01-22T17:59:26Z"
  operationHistory:
    deprovision:
      end: null
      start: null
    inspect:
      end: "2021-01-22T17:59:25Z"
      start: "2021-01-22T17:54:58Z"
    provision:
      end: null
      start: null
    register:
      end: "2021-01-22T17:54:58Z"
      start: "2021-01-22T17:54:48Z"
  operationalStatus: OK
  poweredOn: true
  provisioning:
    ID: d93b973e-2c7f-40d4-be94-83fad34d1257
    bootMode: legacy
    image:
      url: ""
    state: ready
  triedCredentials:
    credentials:
      name: node-0-bmc-secret
      namespace: metal3
    credentialsVersion: "1775"
```

</details>

### What's in the Spec

* `spec.bootMACAddress` - is the MAC address of a host we want to provision.
* `spec.bmc` - containes `address` field where we can specify the Baseboard Management Controller
(BMC) address, while `credentialsName` is a reference to a Kubernetes [secret][secret] object
which stores host's BMC credentials.

You can see the full API description [here][api description].

## Deploy Baremetal Operator

In this section we are going to cover how to quickly deploy the operator into your
Kubernetes cluster.

1. To deploy the operator you need a Kubernetes cluster. You can use [Kind][Kind] tool to create a
cluster. Please follow the instuctions [here][kind-instructions] on how to install the Kind and
[here][create-cluster] on how to create a Kind cluster.

1. We will use [Kustomize][Kustomize] to patch our configs before applying them. You can
  follow the instructions [here][install-kustomize] on how to install the Kustomize. If you
  have Kustomize already installed, skip this step.

1. Create `baremetal-operator-system` namespace where the operator will be running.

1. Deploy `BareMetalHost` Custom Resource Defition (CRD), RBAC rules and the manager to
  run the operator.

    ```shell
    git clone https://github.com/metal3-io/baremetal-operator.git
    cd baremetal-operator
    kustomize build config/default | kubectl apply -f -
    ```

1. Once you have installed CRD and the operator, you can go ahead and create `BareMetalHost`
   object from [example-host.yaml][example-host]. The example-host.yaml includes a `Secret` to store the
   BMC credentials and the `BareMetalHost`. Before applying them, make sure to update the `Secret`
   with the actual username/password and `spec.bmc.address` in BareMetalHost with your host's
   actual BMC address.

    ```shell
    kubectl apply -f https://raw.githubusercontent.com/metal3-io/baremetal-operator/master/examples/example-host.yaml
    ```

    At this point, we have the operator running in our cluster, watching for changes on the custom resource `BareMetalHost`.

1. The next step is to run Ironic service before we move on to provision the host. We will run a couple of Ironic containers
   each serving different purposes. In the next steps, whenever we say Ironic, we will be refering to the below list of containers.

   * ironic-inspector-log-watch
   * ironic-inspector
   * ironic-log-watch
   * ironic-conductor
   * ironic-api
   * mariadb
   * httpd
   * dnsmasq
   * httpd-infra

   There are two ways to run the Ironic.

    1. Outside of the cluster. You can use [run_local_ironic.sh][run_local_ironic.sh] helper script to spin up Ironic containers.

        ```shell
        ./tools/run_local_ironic.sh
        ```

    2. Within the cluster as a deployment.

        ```shell
        kustomize build ironic-deployment/default/ | kubectl apply  -f -
        ```

[Kubernetes operator pattern]: https://kubernetes.io/docs/concepts/extend-kubernetes/operator/2w
[Ironic]: https://docs.openstack.org/ironic/latest/
[secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[Kind]: https://github.com/kubernetes-sigs/kind
[kind-instructions]: https://kind.sigs.k8s.io/docs/user/quick-start/#installation
[create-cluster]: https://kind.sigs.k8s.io/docs/user/quick-start/#creating-a-cluster
[Kustomize]: https://kustomize.io/
[install-kustomize]: https://kubectl.docs.kubernetes.io/installation/kustomize/
[example-host]: https://raw.githubusercontent.com/metal3-io/baremetal-operator/master/examples/example-host.yaml
[run_local_ironic.sh]: https://github.com/metal3-io/baremetal-operator/blob/master/tools/run_local_ironic.sh
[api description]: https://github.com/metal3-io/baremetal-operator/blob/71eb0b294d4804f2c046aa53c664aaa9eeda6be6/docs/api.md