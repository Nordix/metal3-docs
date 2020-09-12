# Metal3DataTemplate

## Metal3DataTemplate example

```yaml
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
kind: Metal3DataTemplate
metadata:
  name: nodepool-1
  namespace: default
  ownerReferences:
  - apiVersion: infrastructure.cluster.x-k8s.io/v1alpha4
    controller: true
    kind: Metal3Cluster
    name: cluster-1
spec:
  metaData:
    strings:
      - key: abc
        value: def
    objectNames:
      - key: name_m3m
        object: metal3machine
      - key: name_machine
        object: machine
      - key: name_bmh
        object: baremetalhost
    indexes:
      - key: index
        offset: 0
        step: 1
    ipAddressesFromIPPool:
      - key: ip
        Name: pool-1
    prefixesFromIPPool:
      - key: ip
        Name: pool-1
    gatewaysFromIPPool:
      - key: gateway
        Name: pool-1
    dnsServersFromIPPool:
      - key: dns
        Name: pool-1
    fromHostInterfaces:
      - key: mac
        interface: "eth0"
    fromLabels:
      - key: label-1
        object: machine
        label: mylabelkey
    fromAnnotations:
      - key: annotation-1
        object: machine
        annotation: myannotationkey
  networkData:
    links:
      ethernets:
        - type: "phy"
          id: "enp1s0"
          mtu: 1500
          macAddress:
            fromHostInterface: "eth0"
        - type: "phy"
          id: "enp2s0"
          mtu: 1500
          macAddress:
            fromHostInterface: "eth1"
      bonds:
        - id: "bond0"
          mtu: 1500
          macAddress:
            string: "XX:XX:XX:XX:XX:XX"
          bondMode: "802.1ad"
          bondLinks:
            - enp1s0
            - enp2s0
      vlans:
        - id: "vlan1"
          mtu: 1500
          macAddress:
            string: "YY:YY:YY:YY:YY:YY"
          vlanId: 1
          vlanLink: bond0
    networks:
      ipv4DHCP:
        - id: "provisioning"
          link: "bond0"

      ipv4:
        - id: "Baremetal"
          link: "vlan1"
          IPAddressFromIPPool: pool-1
          routes:
            - network: "0.0.0.0"
              netmask: 0
              gateway:
                fromIPPool: pool-1
              services:
                dns:
                  - "8.8.4.4"
                dnsFromIPPool: pool-1
      ipv6DHCP:
        - id: "provisioning6"
          link: "bond0"
      ipv6SLAAC:
        - id: "provisioning6slaac"
          link: "bond0"
      ipv6:
        - id: "Baremetal6"
          link: "vlan1"
          IPAddressFromIPPool: pool6-1
          routes:
            - network: "0::0"
              netmask: 0
              gateway:
                string: "2001:0db8:85a3::8a2e:0370:1"
              services:
                dns:
                  - "2001:4860:4860::8844"
                dnsFromIPPool: pool6-1
    services:
      dns:
        - "8.8.8.8"
        - "2001:4860:4860::8888"
status:
  indexes:
    "0": "machine-1"
  dataNames:
    "machine-1": nodepool-1-0
  lastUpdated: "2020-04-02T06:36:09Z"
```

## Metal3DataTemplate spec.metaData

#### strings

Renders the given string as value in the metadata. It takes a
`value` attribute.

#### objectNames

Renders the name of the object that matches the type given.
It takes an `object` attribute, containing the type of the object.

#### indexes

Renders the index of the current object, with the offset from the
`offset` field and using the step from the `step` field. The following
conditions must be matched : `offset` >= 0 and `step` >= 1
if the step is unspecified (default value being 0), the controller will
automatically change it for 1. The `prefix` and `suffix` attributes are to
provide a prefix and a suffix for the rendered index.

#### ipAddressesFromIPPool

Renders an ip address from an *IPPool*
object. The *IPPool* objects are defined in the
[IP Address manager repo](https://github.com/metal3-io/ip-address-manager)

#### prefixesFromIPPool

Renders a network prefix from an *IPPool*
object. The *IPPool* objects are defined in the
[IP Address manager repo](https://github.com/metal3-io/ip-address-manager)

#### gatewaysFromIPPool

Renders a network gateway from an *IPPool*
object. The *IPPool* objects are defined in the
[IP Address manager repo](https://github.com/metal3-io/ip-address-manager)

#### dnsServersFromIPPool

Renders a dns servers list from an *IPPool*
object. The *IPPool* objects are defined in the
[IP Address manager repo](https://github.com/metal3-io/ip-address-manager)

#### fromHostInterfaces

Renders the MAC address of the BareMetalHost that
matches the name given as value.

#### fromLabels

Renders the content of a label on an object or an empty string
if the label is absent. It takes an `object` attribute to specify the type of
the object where to fetch the label, and a `label` attribute that contains the
label key.

#### fromAnnotations

Renders the content of a annotation on an object or an
empty string if the annotation is absent. It takes an `object` attribute to
specify the type of the object where to fetch the annotation, and an
`annotation` attribute that contains the annotation key.

For each object, the attribute **key** is required.

### Metal3DataTemplate spec.networkData

The `networkData` field will contain three items :

* **links**: a list of layer 2 interface
* **networks**: a list of layer 3 networks
* **services** : a list of services (DNS)

#### Links specifications

The object for the **links** section list can be:

* **ethernets**: a list of ethernet interfaces
* **bonds**: a list of bond interfaces
* **vlans**: a list of vlan interfaces

The **links/ethernets** objects contain the following:

* **type**: Type of the ethernet interface
* **id**: Interface name
* **mtu**: Interface MTU
* **macAddress**: an object to render the MAC Address

The **links/ethernets/type** can be one of :

* bridge
* dvs
* hw_veb
* hyperv
* ovs
* tap
* vhostuser
* vif
* phy

The **links/ethernets/macAddress** object can be one of:

* **string**: with the desired Mac given as a string
* **fromHostInterface**: with the interface name from BareMetalHost
  hardware details.

The **links/bonds** object contains the following:

* **id**: Interface name
* **mtu**: Interface MTU
* **macAddress**: an object to render the MAC Address
* **bondMode**: The bond mode
* **bondLinks** : a list of links to use for the bond

The **links/bonds/bondMode** can be one of :

* 802.1ad
* balance-rr
* active-backup
* balance-xor
* broadcast
* balance-tlb
* balance-alb

The **links/vlans** object contains the following:

* **id**: Interface name
* **mtu**: Interface MTU
* **macAddress**: an object to render the MAC Address
* **vlanId**: The vlan ID
* **vlanLink** : The link on which to create the vlan

#### The networks specifications

The object for the **networks** section can be:

* **ipv4**: a list of ipv4 static allocations
* **ipv4DHCP**: a list of ipv4 DHCP based allocations
* **ipv6**: a list of ipv6 static allocations
* **ipv6DHCP**: a list of ipv6 DHCP based allocations
* **ipv6SLAAC**: a list of ipv6 SLAAC based allocations

The **networks/ipv4** object contains the following:

* **id**: the network name
* **link**: The name of the link to configure this network for
* **ipAddressFromIPPool**: renders an ip address from an *IPPool*
  object. The *IPPool* objects are defined in the
  [IP Address manager repo](https://github.com/metal3-io/ip-address-manager)
* **routes**: the list of route objects

The **networks/ipv*/routes** is a route object containing:

* **network**: the subnet to reach
* **netmask**: the mask of the subnet as integer
* **gateway**: the gateway to use, it can either be given as a string in
  *string* or as an IPPool name in *fromIPPool*
* **services**: a list of services object as defined later

The **networks/ipv4Dhcp** object contains the following:

* **id**: the network name
* **link**: The name of the link to configure this network for
* **routes**: the list of route objects

The **networks/ipv6** object contains the following:

* **id**: the network name
* **link**: The name of the link to configure this network for
* **ipAddressFromIPPool**: renders an ip address from an *IPPool*
  object. The *IPPool* objects are defined in the
  [IP Address manager repo](https://github.com/metal3-io/ip-address-manager)
* **routes**: the list of route objects

The **networks/ipv6Dhcp** object contains the following:

* **id**: the network name
* **link**: The name of the link to configure this network for
* **routes**: the list of route objects

The **networks/ipv6Slaac** object contains the following:

* **id**: the network name
* **link**: The name of the link to configure this network for
* **routes**: the list of route objects

#### the services specifications

The object for the **services** section can be:

* **dns**: a list of dns service with the ip address of a dns server
* **dnsFromIPPool**: the IPPool from which to fetch the dns servers list

### The generated secrets

The name of the secret will be made of a prefix and the index. The Metal3Machine
object name will be used as the prefix. A `-metadata-` or `-networkdata-` will
be added between the prefix and the index.

## Deployment flow

### Manual secret creation

In the case where the Metal3Machine is created without a `dataTemplate` value,
if the `metaData` or `networkData` fields are set (one or both), the
Metal3Machine reconciler will fetch the secret, set the status field and
directly start the provisioning of the BareMetalHost using the secrets if given.
If one of the secrets does not exist, the controller will wait to start the
provisioning of the BareMetalHost until it exists.

### Dynamic secret creation

In the case where the Metal3Machine is created with a `dataTemplate` value, the
Metal3Machine reconciler will create a *Metal3DataClaim* for that object.

The *Metal3DataClaim* would then be reconciled, and its controller will create
an index for this *Metal3DataClaim* if it does not exist yet, and create a
Metal3Data object with the index. Upon success, it will set the *ready* field
to true, and the *renderedData* field to reference the *Metal3Data* object.

The Metal3Data reconciler will then generate the secrets, based on the index,
the Metal3DataTemplate and the machine. Once created, it will set the status
field `ready` to True.

Once the Metal3Data object is ready, the Metal3Machine controller will fetch
the secrets that have been created (one or both) and use them to start
provisioning the BareMetalHost.

### Hybrid configuration

If the Metal3Machine object is created with a `dataTemplate` field set, but one
of the `metaData` or `networkData` is also set in the spec, this one will
override the template generation for this specific secret. i.e. if the user sets
the three fields, the controller will use the user input secret for both.

This means that some hybrid scenarios are supported, where the user can give
directly the `metaData` secret and let the controller render the `networkData`
secret through the Metal3DataTemplate object.
