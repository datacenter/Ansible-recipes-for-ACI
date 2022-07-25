## Usage
```
ansible-playbook bgp_with_loop.yaml -i inventory.yaml
```


## Simplified Configuration

Ideally, the ansible playbook itself should be the simplified configuration.
But for the sake of learning, this is an even further simplifed configuration to describe this example.

```
L3Out
    + Node Profile
        + pod-1/node-101
            - router_id 1.1.1.1
        + pod-1/node-102
            - router_id 2.2.2.2

        + Interface Profile
            + pod-1/node-101/eth-1/11
                - sub-interface (VLAN 2051)
                - 10.1.0.1/30
                + BGP Peer
                    - 10.1.0.2
                    - remote AS 65003

            + pod-1/node-102/eth-1/11
                - sub-interface (VLAN 2051)
                - 10.1.0.5/30
                + BGP Peer
                    - 10.1.0.6
                    - remote AS 65003

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


## Topology

```
  AS XXXXX                         AS XXXXX
  Router ID: 1.1.1.1               Router ID: 2.2.2.2
  +----------+                     +----------+
  | node-101 |                     | node-102 |
  +----------+                     +----------+
       | e1/11.x                        | e1/11.x
       |   - 10.1.0.1/30                |   - 10.1.0.5/30
       |   - sub-interface              |   - sub-interface
       |   - vlan-2051                  |   - vlan-2051
       |   - mtu 1500                   |   - mtu 1500
       |                                |
       +--------------+   +-------------+
                      |   |
             10.1.0.2 |   | 10.1.0.6
                     (router)
                     AS 65003
                   172.16.1.0/24
                   172.16.2.0/24
                   192.168.1.0/24
```

Note: ACI BGP AS number (XXXXX) is what you configured in MP-BGP, which is not the scope of this playbook.


## More Explanations

### BGP Peer Connectivity Profiles (`cisco.aci.aci_l3out_bgp_peer`)

`cisco.aci.aci_l3out_bgp_peer` creates peering profiles under each interface configured in L3Out interface profiles. This sources BGP sessions from those interfaces.

However, the module does not support a peering profile under L3Out node profiles, which sources BGP sessions from the loopback on each node.


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

