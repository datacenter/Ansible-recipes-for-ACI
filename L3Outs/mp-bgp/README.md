## Usage
```
ansible-playbook mp-bgp.yaml -i inventory.yaml
```

This playbook is equivalent to MP-BGP Route Reflector config in APIC GUI First Time Setup wizard.
This is required to configure MP-BGP session between route reflector spines of your choice and all leaf nodes.


The BGP AS number configured in this playbook becomes the AS number of all BGP L3Outs.



## More Explanations

### Ansible module `aci_rest`

Ansible collection `cisco.aci` does not support the following two:

* Fabric Pod Profile
* Fabric Pod Policy Group

These are required to apply the BGP route reflector config to the fabric.


Because of this we are using `cisco.aci.aci_rest` to send REST API via Ansible for these as a workaround.  `./templates` folder includes REST API JSON templates for these two configurations.
