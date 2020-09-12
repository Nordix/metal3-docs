# IPAddress

## IPAddress example

```yaml
apiVersion: ipam.metal3.io/v1alpha1
kind: IPAddress
metadata:
  name: pool-1-192-168-0-11
  namespace: default
  ownerReferences:
  - apiVersion: ipam.metal3.io/v1alpha1
    kind: IPPool
    name: pool-1
  - apiVersion: ipam.metal3.io/v1alpha1
    kind: IPClaim
    name: RenderedData-1
spec:
  claim:
    Name: RenderedData-1
  Address: 192.168.0.11
  prefix: 24
  gateway: 192.168.0.1
  dnsServers:
    - 8.8.8.8
  pool:
    Name: pool-1
status:
  ready: true
```

## IPAddress spec


## IPAddress status
