# Metal3Data

The output of the controller would be a Metal3Data object,one per node linking to the
Metal3DataTemplate object and the associated secrets

## Metal3Data example

```yaml
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
kind: Metal3Data
metadata:
  name: nodepool-1-0
  namespace: default
  ownerReferences:
  - apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
    controller: true
    kind: Metal3DataTemplate
    name: nodepool-1
spec:
  index: 0
  metaData:
    name: machine-1-metadata
    namespace: default
  networkData:
    name: machine-1-metadata
    namespace: default
  metal3Machine:
    name: machine-1
    namespace: default
status:
  ready: true
  error: false
  errorMessage: ""
```

## Metal3Data spec

The Metal3Data contains the index of this node, and links to the secrets
generated and to the Metal3Machine using this Metal3Data object.

If the Metal3DataTemplate object is updated, the generated secrets will not be
updated, to allow for reprovisioning of the nodes in the exact same state as
they were initially provisioned. Hence, to do an update, it is necessary to do
a rolling upgrade of all nodes.

The reconciliation of the Metal3DataTemplate object will also be triggered by
changes on Metal3Machines. In the case that a Metal3Machine gets modified, if
the `dataTemplate` references a Metal3DataTemplate, that *Metal3DataClaim*
object will be reconciled. There will be two cases:

* An already generated Metal3Data object exists for that *Metal3DataClaim*. If
  the reference is not in the *Metal3DataClaim* object, the reconciler will add
  it. The reconciler will also verify that the required secrets exist. If they
  do not, they will be created.
* if no Metal3Data exists for that *Metal3DataClaim*, then the
  reconciler will create one and fill the respective field with the secret name.

To create a Metal3Data object, the *Metal3DataClaim* controller will select an
index for that Metal3Machine. The selection happens by selecting the lowest
available index that is not in use. To do that, the controller will list all
existing Metal3Data object linked to this Metal3DataTemplate and to get the
unavailable indexes. The indexes always start from 0 and increment by 1. The
lowest available index is to be used next. The `dataNames` field contains the
map of Metal3Machine to Metal3Data and the `indexes` contains the map of
allocated indexes and claims.

Once the next lowest available index is found, it will create the Metal3Data
object. The name would be a concatenation of the Metal3DataTemplate name and
index. Upon conflict, it will fetch again the list to consider the new list of
Metal3Data and try to create the new object with the new index, this will happen
until the new object is created successfully. Upon success, it will render the
content values, and create the secrets containing the rendered data. The
controller will generate the content based on the `metaData` or `networkData`
field of the Metal3DataTemplate Specs. The *ready* field in *renderedData* will
then be set accordingly. If any error happens during the rendering, an error
message will be added.
