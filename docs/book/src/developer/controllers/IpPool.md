# IpPool

## IpPool example

```yaml
apiVersion: ipam.metal3.io/v1alpha1
kind: IPPool
metadata:
  name: pool-1
  namespace: default
  ownerReferences:
  - apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
    kind: Metal3Cluster
    name: cluster-1
spec:
  clusterName: cluster-1
  pools:
    - start: 192.168.0.10
      end: 192.168.0.15
      subnet: 192.168.0.0/24
      gateway: 192.168.0.1
      prefix: 24
      dnsServers:
        - 8.8.8.8
    - start: 192.168.1.10
      end: 192.168.1.15
      subnet: 192.168.1.0/24
      gateway: 192.168.1.1
      prefix: 24
  gateway: 192.168.1.1
  prefix: 24
  dnsServers:
    - 8.8.4.4
  preAllocations:
    "RenderedData-10": 192.168.0.9
    "RenderedData-9": 192.168.0.8
  namePrefix: "provisioning"
status:
  lastUpdated: "2020-04-02T06:36:09Z"
  allocations:
    "RenderedData-1": "192.168.0.11"
    "RenderedData-10": 192.168.0.9
    "RenderedData-9": 192.168.0.8
```

## IpPool spec

#### pools

A list that contains a list of IP address pools.

#### pools.start

Start IP addresses of the pool. Specifying single ip addresses
can be achieved by setting the start and end ip address to that 
single ip address.

#### pools.end

End IP addresses of the pool.

#### pools.subnet

A field that allows to verify that the allocated IP is in the 
pool and from which the start and end ip addresses can be inferred.

#### prefix

A prefix length from a network in Classless Inter-Domain 
Routing (CIDR) notation. It can be given for each pool of
the list, or globally. If given for a pool it will override
the global setting, that is default value. It will be set
on the `IPAddress` and can be fetched from a Template.

#### dnsServers

An IP address of a dns server. It can be given for each
pool of the list, or globally. If given for a pool it
will override the global setting, that is default value.
It will be set on the `IPAddress` and can be fetched
from a Template.

#### gateway

An IP address of a network gateway. It can be given for
each pool of the list, or globally. If given for a pool
it will override the global setting, that is default value.
It will be set on the `IPAddress` and can be fetched from
a Template.

#### preAllocations

A map of object name and ip address that allow a
user to specify a set of static allocations for some objects.

#### namePrefix

It contains the prefix used to name the IPAddress objects created.
It must remain the same for a subnet, across updates or changes in the
`IPPool` object to keep the existing leases.

## IpPool status

#### lastUpdated

A field with the timestamp of the last update.

> **Note:** In case of an error during the allocation (pool exhaustion for example),
> the error would be reported on the `Claim` object, in the `errorMessage` field.
> The allocations map will map the IP address to the `RenderedData` object it was
> allocated for and the addresses will map the `RenderedData` objects with the
> `IPAddress` objects.