# Metal3Cluster Controller 

The metal3Cluster object contains information related to the deployment 
of the cluster on Baremetal. 

```
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

```controlPlaneEndpoint:``` is an endpoint used to communicate with the 
target’s cluster apiserver.

```noCloudProvider:``` - is a boolen type and indicates whether the cluster 
will be deployed with an external cloud provider or not. If set to ```true```,
Cluster-api-provider-metal3 will patch the target cluster node objects to add 
a providerID. This allows Cluster API process to continue even if the cluster 
is deployed without a cloud provider.
