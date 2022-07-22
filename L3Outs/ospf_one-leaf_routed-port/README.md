## Usage
```
ansible-playbook ospf_one-leaf_routed-port.yaml -i inventory.yaml
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

        + Interface Profile
            + pod-1/node-101/eth-1/12
                - 10.2.0.1/30

            + OSPF I/F Profile
                + OSPF Policy [p2p]
                    - network type: point-to-point

    + External EPG 1
        + 0.0.0.0/0
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
  Router ID: 1.1.1.1
  +----------+
  | node-101 |
  +----------+
       | e1/12
       |   - 10.2.0.1/30
       |   - routed port
       |   - OSPF point-to-point
       |   - mtu 1500
       |
   (router)
  172.16.1.0/24
```


## More Explanations

### Ansible module `aci_rest`

Ansible collection `cisco.aci` does not support the following two:

* OSPF External Policy (area id, area type)
* OSPF Interface Profile (network type)

Because of this we are using `cisco.aci.aci_rest` to send REST API via Ansible for these as a workaround.  `./templates` folder includes REST API JSON templates for these two configurations.

