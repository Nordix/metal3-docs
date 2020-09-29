# Metal3MachineTemplate

The metal3MachineTemplate contains the template to create Metal3Machine

## Metal3MachineTemplate example

```yaml
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
kind: Metal3MachineTemplate
metadata:
  name: md-0
spec:
  template:
    spec:
      image:
        url: https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
        checksum: https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img.md5sum
      hostSelector:
        matchLabels:
          key1: value1
        matchExpressions:
          key: key2
          operator: in
          values: {‘abc’, ‘123’, ‘value2’}
      dataTemplate:
        Name: md-0-metadata
```

## Metal3MachineTemplate spec

## Metal3MachineTemplate status
