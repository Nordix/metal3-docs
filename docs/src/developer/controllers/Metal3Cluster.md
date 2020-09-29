# Metal3Cluster

The metal3Cluster object contains information related to the deployment of the cluster on baremetal. 
It currently has two specification fields:

## Metal3Cluster example

```yaml
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
kind: Metal3Cluster
metadata:
  name: example_cluster
spec:
  controlPlaneEndpoint:
    host: 192.168.111.249
    port: 6443
noCloudProvider: true
```

## Metal3Cluster spec

`controlPlaneEndpoint` - is an endpoint used to communicate
with the target’s cluster apiserver.

`noCloudProvider` - is a boolean. As there is no cloud provider
behind Metal³, this field is usually set to true, meaning that no
external cloud provider will be used to deploy the cluster on.
