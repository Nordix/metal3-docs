# Introduction

The purpose of this document is documenting observations made during the upgrade
process of various kubernetes components using the metal3-dev-env environment.
The following list contains the experiments performed and their outcomes. The
list could grow or shrink overtime as we learn more about the process.

 If you have use cases to be covered, observations of your own or any input,
 please do create a pull request with content in the relevant sections or a
 section of your own.

As to help others to perform the same experiments, please do divide your work
into sections as shown in the ```Outcomes and observations```. Briefly, it should
include the versions of different kubernetes components before and after the
upgrade process. Also relevant is documenting environmental issues noticed during
the experiments, such as delays of some operations.

# Experiments

We cover different cases of upgrade. Broadly speaking, we focus on the following
cases and add sub cases as needed.

- Upgrade of kubernetes version
- Upgrade of kubernetes binaries, kubeadm, kubelet and kubectl
- Upgrade of boot disk image
- Upgrade of a single master
  - Version 1 -> Version 2
  - Version 1 -> Version 2 -> Version 3
- Upgrade of multiple masters
  - Upgrade of masters with one extra Server
  - Upgrade of masters with no extra server
- Upgrade using kubeadmcontrolplanes resource
- Upgrade Using Machine resource
- Check tolerance of number of nodes during rolling update (plus/minus)
- Repeat relevant experiments from the above list using worker nodes

# Outcomes and observations

# Case 1: Upgrading Operating System image

```
Task:
  All other variables being constant, change the Operating System image of a master

Methods:
  See the alternatives listed below

Verification:
  A workload on the original master node should migrate to the new one.
```

In all the methods below, once you have provisioned the first master, please do
create a work load for testing the upgrade process.

```bash
# get the node name the is provisioned
kubectl get bmh -n metal3
# get the IP address
sudo virsh net-dhcp-leases baremetal
# ssh and create the work load. please use the correct IP
ssh ubuntu@IP_ADDRESS

kubectl run upgrade-test-server --image=nginx
kubectl scale deployment upgrade-test-server --replicas 10
```

Method 1:
- Actions:
  - Change OS image on metal3Machine resource.
- Result:
  - Upgrade operation is not triggered. Therefore, this method cannot be used.

Method 2:
- Actions:
  - Update metal3Machine as in Method 1
  - Scale up the nodes to 3 (even number not allowed), and wait for provisioning
  - Scale down the nodes to 1
- Result:
  - ongoing work
  - When scaling up, The 2 new nodes are provisioned sequentially.
  - When scaling down, an arbitrary node is removed. Therefore, this method
  cannot be used.

Method 3:
- Actions:
  - Create a new Metal3MachineTemplate with the new OS and  checksum.
  - Update the KubeadmControlPlane resource to refer to new Metal3MachineTemplate
- Result:
  - successfully upgraded
  - Workload migrated


Notes on Method 3

1. The new is image pulled from the internet, [Downlaod link](https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img)
2. The new checksum can be found [here](c2e57d051229b2d79079a12432a9734c)
3. The check sum is the same as what is already in the local httpd server ()
```
# Run this to get the value
sudo docker exec -it httpd-infra cat /shared/html/images/bionic-server-cloudimg-amd64.img.md5sum
c2e57d051229b2d79079a12432a9734c
```
4. The Metal3MachineTemplate resource needs the ```uid``` of the cluster
```
kubectl get cluster -n metal3 -o yaml | grep uid | head -n 1 | cut -f2 -d:
```
5. A sample Metal3MachineTemplate is shown below
```bash
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: Metal3MachineTemplate
metadata:
  name: REPLACE_THIS_WITH_A_UNIQUE_VALUE
  namespace: metal3
  ownerReferences:
  - apiVersion: cluster.x-k8s.io/v1alpha3
    kind: Cluster
    name: test1
    uid: REPLACE_THIS_VALUE_WITH_THAT_OF_THE_CLUSTER
spec:
  template:
    spec:
      hostSelector: {}
      image:
        checksum: http://172.22.0.1/images/bionic-server-cloudimg-amd64.img.md5sum
        url: https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
```

**Observations**

1. Upgrading may not start immediately, may take between 10 and 20 minutes.
2. Chain upgrading gives unexpected result. Therefore, always de-provision all and
start provisioning.
3. After de-provisioning, if a provisioning takes more than 10 minutes, just start
all over again by running ```make clean && make```
4. Downgrading is not possible. (1.17.0 -> 1.17.3->1.17.0)
5. Scaling up while changing kubernetes version did not work.
---

**do not review below this line yet**

---

## Outcomes and observations

## Case I: Kubernetes version upgrade via kubeadmcontrolplanes

Summary

```
Updated Resource: kubeadmcontrolplanes
kubernetes: from v1.17.0 to v1.17.3

Kuberntes version is changed
kubeadm and kubelet are not affected
```

**Before upgrade**

```
kubeadm: v1.17.3
kubectl: Client: v1.17.3
         Server: v1.17.0
kubelet: v1.17.3
```

**After upgrade**

```
kubeadm: v1.17.3
kubectl: Client: v1.17.3
         Server: v1.17.3    # This has changed
kubelet: v1.17.3
```

Notes:
- It takes a while before starting the upgrade.
- It takes a while before the old server is released.
- If provisioning does not start fast enough, say 10 minutes, then run ```make && and make```
- Unlike previous tests, upgrade works without the need of applying any CNI.


## Case II: Kubernetes version upgrade via Machine

```
Updated Resource: Machine
kubernetes: from v1.17.0 to v1.17.3

Upgrade process is not started as Machine level version change does not have any effect
```

**Before upgrade**

```
kubeadm: v1.17.3
kubectl: Client: v1.17.3 & Server: v1.17.0
kubelet: v1.17.3
```
**After upgrade**

- No upgrade occurs
- No entries in log to indicate failure either

## Case III: Kubernetes version upgrade chain via kubeadmcontrolplanes


```

```
export KUBERNETES_VERSION=1.16.7
- chain test with ```1.16.7 -> 1.17.0 -> 1.17.3```

# Unsorted data from previous experiments

Summary:
1. Upon k8s version change, a new control plane node is provisioned.
2. The new nodes does not join the existing one successfully.
3. The first node will break after a while
TO
make kubeadm and kubelet installation specific enough in ./vm-setup/roles/v1aX_integration_test/templates/controlplane_centos.yaml

verify what version is installed.


resource: KubeadmControlPlane
field: kubernetes version
Change: from 1.17.0 to 1.17.3 [does not have kubeadm with the >= version]
Observations:
  A second node is claimed
  The node is provisioned and sshable
  However, api server is not up and running for the reason shown below
  Deleting the cluster, removes both machines

Note:
- It takes a while for the new changes to be detected.
- The new node is JOINING using /tmp/kubeadm-controlplane-join-config.yaml

```bash
ubuntu@node-0:~$ sudo cat /tmp/kubeadm-controlplane-join-config.yaml
apiVersion: kubeadm.k8s.io/v1beta1
controlPlane:
localAPIEndpoint:
  advertiseAddress: ""
  bindPort: 0
discovery:
bootstrapToken:
  apiServerEndpoint: 192.168.111.249:6443
  caCertHashes:
  - sha256:099e5fb9105b9bc152153f46f0d497a0151dbd3fa49699a6af53c406a312ba80
  token: mg0ob2.2afhbnni1lhijrfo
  unsafeSkipCAVerification: false
kind: JoinConfiguration
nodeRegistration:
kubeletExtraArgs:
  node-labels: metal3.io/uuid=29d86090-03b4-4e28-8a5d-1dbcb962b973
name: 'node-0'
```


# Temporary data

Relevant data from experiments. This needs to be re-organized, and possibly 
re-written.

## Fix for VIP on the highest IP

During upgrade of control plane nodes, the VIP should move from the original 
node to the newly upgraded node. However, since all both nodes have the same 
keepalived priority value, IP address determines which node should take the VIP.

There are multiple solution for this problem. We have listed some of the below, 
with description on why some of them are not applicable in the current context 
even though they solve problem.

### Stop keepalived on the existing master

This requires being inside the original control plane node, which can be done 
in one of two ways. One is using ssh, but ssh access may not be possible.

The other method is to use Daemonset. This would be mean running systemcl 
commands from within a Pod. This is a complex and insecure solution.


###  Increase priority on the new master

Increasing the priority on the new master and restarted keepalived is also
possible. However, this solution has the same limitation as above.

One alternative is editing the KCP resource to increase the priority. However, 
editing a KCP resource is not possible at the moment.

### Use an image with higher priority

Before each node's upgrade, we build a new image with a higher keepalived 
priority value and VIP would migrate to the new node.

This approach has multiple problems. One, it requires maintaining a state.
After each upgrade, we need to save the priority value or build images with
increasing priority. Either way, this will result in nodes not being the same.

If we are to query the keepalived priority of the node with VIP, then we would 
have to use ssh or Daemonset.

### Add a script that monitors API server health end point

This is the preferred method. We add a script the runs forever to monitor the
API server health end point and the keepalived statuses. If both of them are 
not running, we do not do anything. Similarly, if both are running, we do
nothing.

If API server is healthy, but keepalived not running, then we start keepalived.
If the VIP comes to the node, then it will not be a problem since API server 
is up and running.

If API server is not healthy, and keepalived running, then VIP is probably 
attached on the wrong node. In this case we stop the keepalived.

The script and systemd unit files can be passed either via cloud init or 
those files can be baked in boot disk image.

### Using a load balancer

This is also a preferred method, but not applicable in the bare metal
deployment context. But, it needs to revisited when load balancers are
part of the deployment.


## Upgrade features

We cover different cases of upgrade. Broadly speaking, we focus on the following cases and add sub cases as needed. 

Upgrade applies for
- Kubernetes version
- Kubernetes binaries version (kubelet, kubeadm and kubectl)
- Boot disk image version
- Combination of kubernetes and Boot disk image (trigger happens only once)

An added case is downgrading of the above cases.

### Control plane upgrade results

**Upgrade of kubernetes version**

Upgrade of a single master
- Version 1 -> Version 2
- Version 1 -> Version 2 -> Version 3

Upgrade of multiple masters
- Upgrade of masters with one extra Server
- Upgrade of masters with no extra server
- Downgrading kubernetes 

Upgrade of boot disk image
 - what resources and fields should be edited.

Combination of k8s and boot disk image
 - This should trigger only a single upgrade
 
### Worker upgrade results [the f.f. needs further editing]
Upgrade of kubernetes version by using NEW kubeadmConfigTemplate
Resource: MachineDeployment and kubeadmConfigTemplate
Field: spec.template.spec.bootstrap.configRef.name of MachineDeployment
Upgrade of a worker with extra nodes succeeds
Upgrade with NO extra node Triggered, but does not go completion due to other factors

*

  strategy:
    rollingUpdate:
        maxSurge: 0
        maxUnavailable: 1


Upgrade of boot disk image
create new Metal3MachineTemplate, use it in MachineDeployment
with keepalived (similarly as in control plane test), succeeds
  Strategy:
    Rolling Update:
      MaxSurge: 0
      MaxUnavailable: 1

**This needs to be combined with the above**

Worker upgrade results (after sync meeting 1)
Method 1: Upgrade of kubernetes version using machineDeployment
Resource: MachineDeployment
Field: spec.template.spec.version
It triggers an upgrade, but does nothing. 
The version can be any value and still triggers an upgrade. (eg. v3.3.3)
CNI is required for worker upgrade
Upgrade of a worker with extra nodes succeeds
Downgrade of a worker with extra nodes succeeds
Upgrade with NO extra node succeeds by scaling in. And, does not affect KCP node

Method 2: Upgrade of kubernetes version by EDITING existing kubeadmConfigTemplate
Resource: KubeadmConfigTemplate
Field: spec.template.spec.preKubeadmCommands
It sometimes triggers the upgrade process, but does nothing.

Method 3: Upgrade of kubernetes version by using NEW kubeadmConfigTemplate
Resource: MachineDeployment and kubeadmConfigTemplate
Field: spec.template.spec.bootstrap.configRef.name of MachineDeployment
Upgrade of a worker with extra nodes succeeds
Downgrade of a worker with extra nodes succeeds
Upgrade with NO extra node succeeds by scaling in with the following values in the machineDeployment resource. Workers are upgraded one by one and the control plane node is not affected.


## sample table
| Tables         | resource/field|result    |comment  |
| --------------:|:-------------:|:--------:|:-------:|
| single master  | KCP           | success |comment1 |
| multiple master| KCP           | success |comment2 |
| downgrade      | KCP           | success |comment3 |
