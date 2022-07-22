## Usage
```
ansible-playbook ospf_two-leafs_vpc.yaml -i inventory.yaml
```


## Simplified Configuration

Ideally, the ansible playbook itself should be the simplified configuration.
But for the sake of learning, this is an even further simplifed configuration to describe this example.

```
L3Out
    - OSPF Area: 0
    - OSPF Area Type: Regular

    + Node Profile
        + pod-1/node-101
            - router_id 1.1.1.1
        + pod-1/node-102
            - router_id 2.2.2.2

        + Interface Profile
            + pod-1/node-101-102/VPC1_IFPG
                - VLAN 2052
                - side A: 10.2.0.1/24
                - side B: 10.2.0.2/24

            + OSPF I/F Profile
                + OSPF Policy [bcast]
                    - network type: broadcast

    + External EPG 1
        + 172.16.1.0/24
            - External Subnets for the External EPG
        + 172.16.2.0/24
            - External Subnets for the External EPG
        + Contracts
            - ICMP (provided)
            - HTTP (consumed)

    + External EPG 2
        + 192.168.1.0/24
            - External Subnets for the External EPG
        + Contracts
            - ICMP (provided)
```
Note:
The following non-L3Out components are not created by this
playbook and need to be created separately.
* Access Policies (Interface profiles, Interface Policy Group and such)
* A L3Out Domain with an AEP and a VLAN pool
* Contracts


## Topology

```
  Router ID: 1.1.1.1               Router ID: 2.2.2.2
  +----------+                     +----------+
  | node-101 |                     | node-102 |
  +----------+                     +----------+
       | e1/2 (VPC1_IFPG)               | e1/2 (VPC1_IFPG)
       |   - 10.2.0.1/24                |   - 10.2.0.2/24
       |   - SVI                        |   - SVI
       |   - vlan-2052                  |   - vlan-2052
       |   - OSPF broadcast             |   - OSPF broadcast
       |   - mtu 1500                   |   - mtu 1500
       |                                |
       +---------------++---------------+
                       ||  <-- vPC
                       ||
                    (router)
                  172.16.1.0/24
                  172.16.2.0/24
                  192.168.1.0/24
```


## More Explanations

### Ansible module `aci_rest`

Ansible collection `cisco.aci` does not support the following two:

* OSPF External Policy (area id, area type)
* OSPF Interface Profile (network type)

Because of this we are using `cisco.aci.aci_rest` to send REST API via Ansible for these as a workaround.  `./templates` folder includes REST API JSON templates for these two configurations.


### Complex loop with the `subelements` filter


In this example, we need to iterate over external EPGs first, then, iterate over external subnets under each external EPG.

This could be achieved by creating two separate variables, one for each loop, like this.

```
l3out:
  extepgs:
    - name: 'EPG1'
    - name: 'EPG2'

  extsubnets:
    - epg: 'EPG1'
      subnet: '172.16.1.0/24'
      scope: ['import-security']
    - epg: 'EPG1'
      subnet: '172.16.2.0/24'
      scope: ['import-security']
    - epg: 'EPG2'
      subnet: '192.168.1.0/24'
      scope: ['import_security']
```

However, this is not efficient and will become hard to maintain as it duplicates some parameters like EPG name.
Some may think the second one `extsubnets` can be used for both even if it means EPG1 is created twice because Ansible will simply say it already exists.
But the problem with that approach is that, if each EPG has other attributes like `preferred_group`, you will need to make sure the same value is configured for the same EPG in all duplicated elements, which is quite error-prone.

Instead, in this playbook, we have variables with nested lists such as below:
```yaml
l3out:
  ...snip...
  extepgs:
    - name: 'EPG1'
      networks:
        - subnet: '172.16.1.0/24'
          scope: ['import-security']
        - subnet: '172.16.2.0/24'
          scope: ['import-security']
      ...snip...
    - name: 'EPG2'
      networks:
        - subnet: '192.168.1.0/24'
          scope: ['import_security']
      ...snip...
```

When we create EPGs for which we need each EPG name, we simply loop with `'{{ l3out.extepgs }}'`.


When we create subnets for which we need each subnet and its parent EPG name, we loop with `'{{ l3out.extepgs | subelements("networks") }}'`.

Check this [Ansible doc](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#combining-objects-and-subelements) for the official explanation of the `subelements` filter.

Here is another example to understand how `subelements` works.

1. Original list
```yaml
original_list:
  - keyA: value0
    keyB: nested_list0
  - keyA: value1
    keyB: nested_list1
```

2. Apply subelements with `keyB`
```
'{{ original_list | subelements("keyB") }}'
```

3. A new list created by subelements on the fly
```yaml
  - 
    - original_list.0
    - nested_list0.0
  - 
    - original_list.0
    - nested_list0.1
  - 
    - original_list.1
    - nested_list1.0
  - 
    - original_list.1
    - nested_list1.1
```
This yaml is equivalent to this:
```
[
    [original_list[0], nested_list0[0]],
    [original_list[0], nested_list0[1]],
    [original_list[1], nested_list1[0]],
    [original_list[1], nested_list1[1]],
]
```

If we loop over this new list, within a loop, we can use `item.1` to access each element of the nested lists while `item.0` can be used to access its parent value.

