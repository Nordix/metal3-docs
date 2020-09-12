# IpClaim

## IpClaim example

```yaml
apiVersion: ipam.metal3.io/v1alpha1
kind: IPClaim
metadata:
  name: RenderedData-1
  namespace: default
  ownerReferences:
  - apiVersion: metadata.metal3.io/v1alpha1
    kind: Data
    name: data-1
spec:
  owner:
    name: RenderedData-1
  pool:
    name: pool-1
status:
  ipAddress:
    name: pool-1-192-168-0-11
  errorMessage: ""
```

## IpClaim spec


## IpClaim status
