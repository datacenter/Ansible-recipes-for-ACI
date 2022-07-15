# Ansible recipes for ACI
## _Simple yet powerful playbooks you can use_

This repo contains a series of Ansible playbooks you can use with [Cisco ACI](https://www.cisco.com/c/en/us/solutions/data-center-virtualization/application-centric-infrastructure/index.html).
The playbooks make use of [native ACI Ansible modules](https://docs.ansible.com/ansible/latest/collections/cisco/aci/index.html#plugin-index) as much as possible.
They are kept as simple as possible to make them easy to understand and reuse in your environment.
Example playbooks you can find here include:

- Complete configuration of interface, interface policies, VLAN pools, domains
- Comprehensive multi-tier tenant configuration
- Configuration of common features such as NTP, DNS, BGP Route Reflector
- etc.

## _How to use this code_

Please check [here](https://github.com/CiscoDevNet/ansible-aci) for installation instructions.
You'll need Ansible 2.9 or higher and the Cisco ACI collection installed. Installing the collection is just one instruction.
Inside each directory you'll find a minimal inventory file with one APIC host, its username and password.

## _Optimizations_

To keep the code as simple and light as possible, most playbooks here use regular username/password authencation.
If you'd like to optimize the speed of execution and augment security, consider using [signature-based transactions](https://docs.ansible.com/ansible/latest/scenario_guides/guide_aci.html)

## _Disclaimer and license_

All code found in this repo is provided as-is and subject to all terms and conditions of [the unlicense](https://unlicense.org/)
