# Metal3DataClaim

The `Metal3DataClaim` object will reference its target `Metal3DataTemplate`
object.

## Metal3DataClaim example

```yaml
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
kind: Metal3DataClaim
metadata:
  name: machine-1
  namespace: default
  ownerReferences:
  - apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
    controller: true
    kind: Metal3Machine
    name: machine-1
spec:
  template:
    name: nodepool-1
status:
  renderedData:
    name: nodepool-1-0
  errorMessage: ""
```

## Metal3DataClaim spec

<!-- Add spec info -->

## Metal3DataClaim status

#### renderedData

A reference to the `Metal3Data` object. 

#### errorMessage

A description of the error.
